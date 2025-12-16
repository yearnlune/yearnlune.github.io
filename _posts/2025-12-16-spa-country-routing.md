---
tags: 
    - cloudfront
    - spa
    - seo
    - routing
title: "SPA 국가별 라우팅 설계"
date: 2025/12/16
author: 김동환
description: CloudFront Functions를 이용한 일본 사용자 분기 처리와 그 과정에서의 선택
disabled: false
categories:
    - AWS
    - Frontend
---

SPA 서비스를 운영하다 보면 SEO를 위해 국가별로 URL 구조를 분리해야 하는 상황이 생긴다. 우리는 일본 시장 진출을 앞두고 `/jp` 경로를 통한 국가별 분기를 구현해야 했다.

## 문제 상황

- 일본에서 접속한 사용자는 `/jp`로 시작하는 URL로 보내야 함
- React 기반 SPA이므로 모든 `/jp/**` 요청은 `/jp/index.html`로 가야 함
- 하지만 정적 파일(js, css, 이미지)은 원본 경로 유지 필요
- 검색엔진이 크롤링할 수 있도록 서버 측에서 처리해야 함

## 선택지

**1. Lambda@Edge**
- 풀 프로그래밍 기능 제공
- Node.js 런타임 사용 가능
- 하지만 비용이 높고 콜드 스타트 존재

**2. CloudFront Functions**
- 매우 빠르고 저렴 (Lambda@Edge 대비 1/6 수준)
- 단순한 JavaScript만 가능
- 1ms 미만 실행 시간 제약

**3. 클라이언트 JavaScript 리다이렉트**
- 가장 간단하지만 SEO에 불리
- 검색엔진 크롤러가 국가 헤더를 제공하지 않을 수 있음

## 실제 선택: CloudFront Functions 두 개로 분리

우리는 CloudFront Functions를 선택했고, 관심사를 분리하기 위해 두 개의 함수로 나눴다.

### CloudFront Function 1: 국가 감지 및 리다이렉트

```javascript
function handler(event) {
    var request = event.request;
    var headers = request.headers;
    var uri = request.uri;

    if (headers['cloudfront-viewer-country'] &&
        headers['cloudfront-viewer-country'].value.toLowerCase() === 'jp' &&
        (uri === '/' || uri === '')) {

        return {
            statusCode: 302,
            statusDescription: 'Found',
            headers: {
                'location': { value: '/jp' }
            }
        };
    }

    return request;
}
```

### CloudFront Function 2: SPA 라우팅 Rewrite

```javascript
function handler(event) {
    var request = event.request;
    var uri = request.uri;

    if (uri.indexOf('/jp') === 0) {
        var extensionPattern = /\.[a-z0-9]+$/i;

        if (!extensionPattern.test(uri)) {
            request.uri = '/jp/index.html';
        }
    }

    return request;
}
```

## 왜 이렇게 판단했는가

### 1. 하나의 함수가 아닌 두 개로 분리한 이유

처음엔 하나의 함수에서 국가 체크와 URI rewrite를 모두 하려 했다. 하지만:

- 국가 감지는 루트 경로(`/`)에서만 필요
- URI rewrite는 모든 `/jp/**` 경로에 필요
- 하나로 합치면 매 요청마다 불필요한 국가 체크 로직이 실행됨

두 함수로 나누면 각자의 책임이 명확하고, 나중에 다른 국가를 추가할 때도 CloudFront Function 1만 수정하면 된다.

### 2. Lambda@Edge가 아닌 CloudFront Functions를 선택한 이유

Lambda@Edge가 더 강력하지만, 우리가 하는 일은:
- 헤더 하나 읽기
- 문자열 비교
- URL 변환

이 정도는 CloudFront Functions로 충분했고, 비용과 레이턴시 측면에서 이득이 컸다.

## 트레이드오프

### 1. 302 리다이렉트 선택

일본 사용자가 `/`에 접속하면 `/jp`로 한 번 더 리다이렉트된다. 301이 아닌 302를 선택한 이유:

- `/`가 `/jp`로 영구 이동한 것이 아님
- 국가에 따라 다르게 동작하는 조건부 리다이렉트
- 검색엔진도 `/`와 `/jp`를 각각 별도로 인덱싱해야 함

### 2. URL 경로와 표시 언어의 분리

서버(CloudFront)는 SEO를 위한 URL 구조만 담당한다. 실제 언어 표시는 클라이언트에서 `navigator.language`와 사용자 선택으로 처리한다.

이 덕분에:
- 일본에 있지만 영어를 쓰는 사용자도 `/jp` URL에서 영어로 볼 수 있음
- 일본 외 지역에서도 수동으로 일본어 선택 가능

트레이드오프: URL 경로와 실제 표시 언어가 불일치할 수 있다. 예를 들어 `/jp`인데 영어로 표시될 수 있다. 하지만 SEO는 URL 기반으로만 동작하므로 문제없다.

## 필요한 CloudFront 설정

두 함수를 Viewer Request 단계에 순서대로 연결:

1. CloudFront Function 1 (국가 감지)
2. CloudFront Function 2 (URI rewrite)

그리고 Cache Policy에 `CloudFront-Viewer-Country` 헤더를 포함시켜야 국가별 캐싱이 제대로 작동한다.

## 현재 결론

이 구조의 핵심은 **관심사 분리**와 **적절한 추상화 수준**이다.

서버(CloudFront Functions)는 SEO를 위한 URL 구조만 담당한다. 실제 언어 표시는 클라이언트에서 처리한다. 이렇게 분리함으로써:
- SEO는 URL 기반으로 동작하므로 검색엔진이 국가별 콘텐츠를 명확히 인덱싱할 수 있다
- 사용자는 URL과 무관하게 원하는 언어를 선택할 수 있다
- 서버 로직은 단순하게 유지되어 비용과 레이턴시를 최소화할 수 있다

두 개의 함수로 나눈 것도 같은 원칙이다. 국가 감지와 URI rewrite를 분리함으로써 각 함수의 책임이 명확해진다. 다른 국가를 추가할 때도 CloudFront Function 1만 수정하면 된다.

완벽한 구조는 아니다. 하지만 현재 우리에게 필요한 만큼만 구현했고, 각 선택의 근거를 설명할 수 있다는 점에서 만족한다.
