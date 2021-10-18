---
tags:
    - javascript
    - js
    - es6
    - spread operator
    - 전개
title: ES6 Spread operator
date: 2021/07/13
author: 김동환
description: ES6 전개 구문
disabled: false
categories:
- javascript
---

# Spread Operator

Spread operator(`...`)는 배열이나 객체를 복사, 확장, 결합 등에 활용할 수 있다. 또한 함수의 인수로도 쉽게 활용할 수 있다.

## apply()

```jsx
// ES6 이전
function printArgs(x, y, z) {
    console.log("x: " + x + ", y: " + y + ", z: " + z);
}
var args = [0, 1, 2];
printArgs.apply(null, args);  // x: 0, y: 1, z: 2
```

```jsx
function printArgs(x, y, z) { 
    console.log(`x: ${x}, y: ${y}, z: ${z}`)
}
const args = [0, 1, 2];
printArgs(...args);  // x: 0, y: 1, z: 2
```

## 배열

### array literals

```jsx
const bc = ['b', 'c'];
const abcde = ['a', ...bc, 'd', 'e'];
console.log(JSON.stringify(abcde));  //["a","b","c","d","e"]
```

### one level deep copy

```jsx
const origin = ['a', 'b', 'c'];
const target = [...origin];

target.push('a`');

console.log(JSON.stringify(origin));  // ["a","b","c"]
console.log(JSON.stringify(target));  // ["a","b","c","a`"]
```

```jsx
const origin = [['a'], ['b'], ['c']];
const target = [...origin];

target[0].push('a`');

console.log(JSON.stringify(origin));  // [["a","a`"],["b"],["c"]]
console.log(JSON.stringify(target));  // [["a","a`"],["b"],["c"]]
```

### concat

```jsx
const abc = ['a', 'b', 'c'];
const def = ['d', 'e', 'f'];

const abcdef = [...abc, ...def];
const defabc = [...def, ...abc];

console.log(`abcdef: ${JSON.stringify(abcdef)}`);
console.log(`defabc: ${JSON.stringify(defabc)}`);
```

## 객체

### fields

```jsx
const ab = {a: 'a', b:'b'};
const primeAb = {a: 'a`', b:'b`'};

const cloneAb = {...ab};
const mergeAb = {...ab, ...primeAb};

console.log(`cloneAb: ${JSON.stringify(cloneAb)}`);  // cloneAb: {"a":"a","b":"b"}
console.log(`mergeAb: ${JSON.stringify(mergeAb)}`);  // mergeAb: {"a":"a`","b":"b`"}
```

# 참고 자료

[전개 구문](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Spread_syntax)