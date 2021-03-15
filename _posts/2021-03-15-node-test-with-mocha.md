---
tags: 
    - node
    - node.js
    - js
    - test
    - mocha
    - chai
    - bdd
title: node.js Test
date: 2021/03/08
author: 김동환
description:  node Test with mocha
disabled: false
categories:
  - node
---
> Mocha는 Node.js에서 사용하는 JavaScript 테스트 프레임웍으로 쉽게 테스트 코드를 작성할 수 있으며, 비동기 테스트가 가능하다. Assertion모듈은 포함되어있지 않아, node에서 제공하는 `assert`를 사용하거나 `Chai` 와 같은 라이브러리를 사용할 수 있다. BDD, TDD 등의 인터페이스를 제공하고 있다.
　
# 설치

```bash
# js
npm install --save-dev mocha

# ts
npm install --save-dev mocha @types/mocha
```
　
# BDD 스키마

```tsx
describe('userService', function () {
    describe('#createUser', function () {
        it('correct request', function (done) {
            //...
            done();
        });

        it('incorrect request', function (done) {
            //...
            done();
        });
    })

    describe('#findAllUser', function () {
        it('correct request', function () {
            //...
        });

        it('incorrect request', function () {
            //...
        });
    });
});

/*

userService
    #createUser
      √ correct request
      √ incorrect request
    #findAllUser
      √ correct request
      √ incorrect request

*/
```

- describe: test suite로 여러 test case를 묶어놓은 집합이라 할 수 있다.
- context: describe의 alias이다.
- it: 하나의 test case이다. done parmeter를 통해서 비동기 함수에 대해서도 테스트 가능하다.

　
```tsx
// HOOKS

before(function() {
  // 블록 안에서 가장 처음 한번 실행된다.
});

after(function() {
  // 블록 안에서 가장 나중에 한번 실행된다.
});

beforeEach(function() {
  // 블록 안에서 테스트마다 먼저 실행된다.
});

afterEach(function() {
  // 블록 안에서 테스트마다 나중에 실행된다.
});

```

　
# 참고문헌

[https://mochajs.org/](https://mochajs.org/)

[https://www.chaijs.com/](https://www.chaijs.com/)