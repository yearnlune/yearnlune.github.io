---
tags:
    - javascript
    - js
    - es6
    - destructuring
title: ES6 Destructuring
date: 2021/07/05
author: 김동환
description: ES6 Destructuring 구문
disabled: false
categories:
- javascript
---

# Destructuring

변수의 선언 형식이 구조화된 객체, 배열을 활용하지 않고 패턴 매칭을 통해 데이터를 바인딩해주는 것을 의미한다. 바인딩에 실패하게 되면 유연하게 `undefined`로 할당된다.

# 비구조화 배열

## 할당

```tsx
// ES6 이전
const array = ["a", "b"];
const x = array[0];
const y = array[1];

console.log(`x: ${x}, y: ${y}`); // x: a, y: b
```

```tsx
const [x, y, z] = ["a", "b"];

console.log(`x: ${x}, y: ${y}, z: ${z}`);  // x: a, y: b, z: undefined
```

## Rest operator

```tsx
const [x, y, ...z] = ["a", "b", "c", "d", "e"];

console.log(`x: ${x}, y: ${y}, z: ${JSON.stringify(z)}`);  // x: a, y: b, z: ["c","d","e"]
```

## 초기화

```tsx
const [x = "A", y, z = "c"] = ["a", "b"];

console.log(`x: ${x}, y: ${y}, z: ${z}`);  // x: a, y: b, z: c
```

## 무시하기

```tsx
const [i, , j] = ["a", "b", "c"];

console.log(`i: ${i}, j: ${j}`);  // i: a, j: c
```

　
# 비구조화 객체

## 할당

```tsx
// ES6 이전
const object = {x: "a", y: "b"};
const x = object.x;
const y = object.y;

console.log(`x: ${x}, y: ${y}`);  // x: a, y: b
```

```tsx
const {x, y, z} = {x: "a", y: "b"};

console.log(`x: ${x}, y: ${y}, z: ${z}`);  // x: a, y: b, z: undefined
```

## Rest operator

```tsx
const {x, y, ...z} = {x: "a", y: "b", z: "c", A: "d", B: "e"};

console.log(`x: ${x}, y: ${y}, z: ${JSON.stringify(z)}`);  // x: a, y: b, z: {"z":"c","A":"d","B":"e"}
```

## 초기화

```tsx
const {x = "A", y, z = "c"} = {x: "a", y: "b"};

console.log(`x: ${x}, y: ${y}, z: ${z}`);  // x: a, y: b, z: c
```

## 변수 이름 변경

```tsx
const {x, y: why, z: how = "c"} = {x: "a", y: "b"};

console.log(`x: ${x}, why: ${why}, how: ${how}`);  // x: a, why: b, how: c
```

## 중첩된 객체

```tsx
const {x, y: {Y: why}, z: how = "c"} = {x: "a", y: {X: "B", Y: "b"}};

console.log(`x: ${x}, why: ${why}, how: ${how}`);  // x: a, why: b, how: c
```

## Computed property name

```tsx
let key = "x";
let { [key]: who } = { x: "X" };

console.log(`who: ${who}`);  // who: X
```

# 참고 자료

[구조 분해 할당](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)