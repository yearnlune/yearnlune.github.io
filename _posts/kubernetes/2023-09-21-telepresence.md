---
tags: 
    - telepresence
    - k8s
    - intercept
title: Telepresence을 활용한 개발 환경 구축하기
date: 2023/09/21
author: 김동환
description: K8S 환경에서 Telepresence를 통한 쉽고 간편한 개발하기
disabled: false
categories:
  - general
---

# K8S통한 MSA환경에서 개발

K8S 환경에서 애플리케이션을 개발하고 디버깅하는 것은 쉽지 않다. 각 서비스가 독립적으로 운영되고 각종 서비스나 third-party와 상호작용이 필요 하기 때문이다. 그리하여, 로컬 머신에서 개발하기 위해 여러 가지 방법을 사용한다.

## 클러스터 서비스의 노출 및 접근

클러스터에서 작동되고 있는 서비스를 외부에 노출해 로컬 서비스에서 endpoint를 통해 접근을 할 수 있다.


**로컬 서비스에서 클러스터 서비스에 접근하는 방법**

- NodePort 또는 LoadBalancer를 통한 포트 터널링
- External IP를 통한 노출
- kubectl port-forward


**클러스터 서비스에서 로컬 서비스에 접근하는 방법**

- k8s endpoints의 주소 변경


## 클러스터 환경 구축

로컬 환경에 실제 클러스터 환경에 구축된 각종 서비스와 third-party를 구축하여 클러스터 환경과 같게 만들어 로컬 개발 환경을 만들 수 있다.

- [minikube](https://minikube.sigs.k8s.io/) 및 docker을 통한 환경 구축


# Telepresence

클러스터에 있는 서비스를 로컬 환경에서 손쉽게 개발 및 디버깅을 할 수 있도록 도와주는 도구이다. 종속된 다른 서비스와 third-party에 대한 노출 및 실행 없이 로컬에서 서비스한 것처럼 변경해준다.

![telepresence](/assets/images/telepresence/telepresence.png)


# 따라 해보기

## 개발 환경

- Windows 10
- Telepresence 2.15.0
- k8s 1.19.14


## 다운로드 하기

```powershell
# https://app.getambassador.io/download/tel2/windows/amd64/latest/telepresence.zip
curl -fL https://app.getambassador.io/download/tel2/windows/amd64/latest/telepresence.zip -o telepresence.zip
```


## 로컬 머신에 설치 하기

위에 다운로드 받은 zip파일을 풀어 **관리자 모드 PowerShel**l로 `install-telepresence.ps1` 스크립트를 실행시켜 설치한다.

```powershell
# 실행 정책을 변경
PS> Set-ExecutionPolicy Bypass -Scope Process

# 스크립트 실행
PS> .\install-telepresence.ps1

# 설치 확인하기
PS> telepresence version

# Client     : v2.15.1
# Root Daemon: not running
# User Daemon: not running
```


## 로컬 머신에 k8S 클러스터 설정

[kubectl을 로컬에 설치](https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-windows/)하고, 해당 클러스터의 접근을 위한 설정을 한다.

```bash
# 서버 주소 및 cert 설정
kubectl config set-cluster dev --server=https://kubernetes.default --certificate-authority={path}

# context 설정 - 클러스터
kubectl config set-context dev --cluster=dev 

# 유저 설정
kubectl config set-credentials user --token={token}

# context 설정 - 유저
kubectl config set-context dev --user=user

# dev context 사용
kubectl config use-context dev 
```


## Telepresence connect

telepresence와 연결을 하게 되면 해당 클러스터의 서비스와 연결할 수 있다.

```bash
# telepresence 연결하기
telepresence connect

# 연결 확인하기
curl -ik https://kubernetes.default

# 클러스터 서비스 연결 확인하기
# curl -ik {Cluster service IP}
curl -ik http://10.244.222.49:8080
```


## Telepresence intercept

클러스터의 서비스를 로컬 머신의 서비스로 가로채어 대체할 수 있다. 기본은 Global intercept 형식으로 작동하게 되는데 모든 API 요청을 가로챈다. 그렇기에 여러 개발자가 사용하기 위해선 Private intercept(유료)를 사용해야 한다.

```bash
# telepresence 인터셉트 하기
# telerpesence intercept {service name} --port {local-service Port}:{cluster-service Port}
telepresence intercept example-service --port 8081:8080

# telepresence 인터셉트 해제하기
telepresence uninstall example-service --agent 
```


# 참고 문헌

[Telepresence Quick Start](https://www.telepresence.io/docs/latest/quick-start/)

[How to intercept](https://www.getambassador.io/docs/telepresence/latest/howtos/intercepts)

[Telepresence2](https://livewyer.io/blog/2021/04/26/telepresence-2-showcase-open-source-development-workflows/)

[Telepresence Github](https://github.com/telepresenceio/telepresence)