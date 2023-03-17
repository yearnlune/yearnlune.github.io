---
tags:
- eslint
- eslint config
- eslintrc
title: eslint configuration
date: 2023-03-17
author: 김동환
description: eslint config property
disabled: false
categories:
- javascript
---

# root

보통 eslint는 `.eslintrc.*`를 찾기 위해 상위 폴더로 거슬러 올라가며 찾습니다. 이 옵션의 경우, 끝을 선언하는 것으로 해당 값이 `true`인 경우 더 이상 상위 폴더를 검색하지 않습니다.

```jsx
{
  root: true
}
```

# env

실행환경에 따라 쓰여지는 전역변수가 다릅니다. 전역변수는 따로 선언하지 않고 사용하기에 eslint에선 이를 오류로 판단합니다. 그렇기에 그러한 환경을 우선 등록하여 린팅 오류를 내는 것을 방지할 수 있습니다.

```jsx
{
  env: {
    browser: true,
    node: true,
    jest: true,
  }
}
```

# parser

eslint는 기본적으로 ECMAScript 5 코드를 처리하는 `Espree`를 사용합니다. 하지만 요즘 개발자들은 타입스크립트, JSX 등을 사용해 개발하는 경우가 많아졌습니다. 이러한 코드들을 린팅할 수 없어서 parser를 사용해야 합니다.

> 💡 Code → 파서를 통한 변환 → AST(Abstract Syntax Tree) → 린팅
>

```jsx
{
  parser: '@typescript-eslint/parser'
}
```

## 종류

### @typescript-eslint/parser

타입스크립트 코드를 린팅할 수 있게 해주는 파서

### @babel/eslint-parser

`babel`를 활용하여 변환한 코드를 린팅할 수 있게 해주는 파서

### vue-eslint-parser

`.vue` 파일들을 린팅할 수 있게 해주는 파서

## ParserOptions

파서의 옵션을 설정할 수 있습니다. 기본적으로 ECMAScript 버전이나 JSX 지원 여부를 설정할 수 있습니다. 또한 각 커스텀 파서의 옵션도 설정할 수 있습니다. 해당 옵션들은 각 커스텀 파서들의 문서를 참고해야 합니다.

```jsx
/* Espree */
// https://eslint.org/docs/latest/use/configure/language-options#specifying-parser-options
{ 
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
    ecmaFeatures: {
      jsx: true
    }
  }
}

/* @typescript-eslint/parser */
// https://typescript-eslint.io/architecture/parser/#configuration
{ 
  parserOptions: {
    ecmaVersion: 'latest',
    project: ['./tsconfig.json']
  }
}

/* vue-eslint-parser*/
// https://github.com/vuejs/vue-eslint-parser#readme
{ 
  parserOptions: {
    ecmaVersion: 'latest',
    parser: '@typescript-eslint/parser',
    sourceType: 'module',
  }
}
```

# plugins

eslint에서 기본으로 제공하는 규칙외에도, 다양한 규칙들을 서드파티 플러그인을 통해 쉽게 적용할 수 있습니다. `eslint-plugin-` 접두사는 생략할 수 있다. 해당 플러그인을 선언 하더라도 그 플러그인의 규칙들이 적용되는 것은 아닙니다. 해당 플러그인을 선언하고 `rules` 에서 세세한 규칙들을 적용합니다.

```jsx
{
  plugins: [
    'vue',
    'prettier'
  ]
}
```

## 종류

### prettier

포맷터인 `prettier`를 eslint의 규칙으로 동작하게 하는 플러그인

### eslint-plugin-vue

vue 린팅을 위한 플러그인

### eslint-plugin-graphql

graphql 린팅을 위한 플러그인

### eslint-plugin-import

import/export의 문법 린팅을 위한 플러그인

# settings

`plugins`의 설정을 선언할 수 있습니다. 각 설정방법은 플러그인의 문서를 참고해야합니다.

```jsx
{
  plugins: [
    'import'
  ],
  settings: {
    'import/resolver': {
      typescript: {},
      node: {
        extensions: ['.js', '.jsx', '.ts', '.tsx'],
      },
    },
  },
}
```

# extends

`sharable config`를 선언하는 곳으로, 기존의 규칙들(ruleset)을 확장하여 적용할 수 있다. `eslint-config-` 접두사는 생략할 수 있습니다. 여러개의 규칙들을 적용할 수 때문에, 중복된 규칙의 경우 다른 규칙들에 의해 덮어쓰일 수 있습니다. 그리하여, 순서를 고려하며 적용해야 합니다. 대부분의 플러그인에서는 기존 `sharable config`도 같이 제공해주기 때문에 `plugins`에 선언을 하지 않고 `plugin:` 접두사(플러그인임을 선언)를 사용하여 쉽게 적용할 수 있습니다.

```jsx
{
  extends: [
    'plugin:vue/vue3-recommended',  // vue 플러그인의 vue3-reocmmended config를 적용한다.
    '@vue/airbnb-with-typescript',
    '@vue/prettier',
  ]
}
```

## 종류

### **eslint-config-airbnb**

`airbnb`에서 사용하고 있는 린팅 설정



# rules

코드를 검사할 때 적용하는 규칙들을 설정할 수 있습니다. 기본적으로 `off(0)`  , `warn(1)` , `error(2)`로 설정할 수 있습니다. `extends`에서 적용된 규칙보다 우선적용합니다.

```jsx
{
  rules: [
    'no-console': 0,
    'prefer-const': 2,
    'no-undef': 2,
  ]
}
```

# overrides

특정 파일들의 eslint를 다르게 적용하고 싶을 때 사용합니다.

```jsx
{
  overrides: [
    {
      'files': ['test/**'],
      'plugins': ['jest'],
      'extends': ['plugin:jest/recommended'],
      'rules': { 'jest/prefer-expect-assertions': 'off' },
    }
  ]
}
```

# 참고문헌

[eslint documentation](https://eslint.org/docs/latest/)


[eslint plugins](https://github.com/dustinspecker/awesome-eslint)