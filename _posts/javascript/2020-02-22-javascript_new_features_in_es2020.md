---
layout: post
title: 'Javascript ES2020 새로운 기능'
subtitle: 'Short, useful Javascript lessons - make it easy'
categories: [javascript, es2020]
image: https://images.unsplash.com/photo-1521185496955-15097b20c5fe?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1247&q=80
featured: true
hidden: false
---

> [원문: JavaScript new features in ES2020](https://medium.com/javascript-in-plain-english/new-javascript-features-in-es2020-c2d76acf9c5a)

ES2020의 최신 기능에 대해서 알아봅시다.

- Private Class Variables
- Promise.allSettled
- String.prototype.matchAll
- Optional Chaining Operator
- Dynamic Import
- BigInt

노트: 모든 예제는 Chrome 버전 79에서 테스트 되었습니다.

## Private Class Variables

`hash(#)`를 변수의 앞이나 함수에 사용하는 것만으로 private 접근제어를 하는 것이 가능해집니다.

```javascript
class Person {
  #born = 1980
  age() { console.log(2020 - this.#born) }
}
const person1 = new Person()
person1.age() // 40
console.log(person1.#age)
//Uncaught SyntaxError: Private field '#age' must be declared in an
//enclosing class
```

## Promise.allSettled

ES6에서 `promise`가 소개된 이후로 Javascript에서는 `Promise.all`, `Promise.race`라는 static methods를 지원해 왔습니다. 이제 `Promise.allSettled`가 ES2020에서 새롭게 추가됩니다.

promise가 하나라도 rejected 되거나 settled 됐을 때 실행을 중단하는 `Promise.all`, `Promise.race`와 다르게 `Promise.allSettled`는 어느 promise의 중단과 상관없이 모든 promise를 실행시킵니다.

- `Promise.allSettled`는 ES2020에서 short-circuit 되지 않습니다.
- `Promise.all`은 ES6에서 입력값이 거절되면 short-circuit 처리됩니다.
- `Promise.race`는 ES6에서 입력값이 거절되면 short-circuit 처리됩니다.

`Promise.allSettled`를 사용하면 모든 Promise가 처리 완료됐을 때만 값을 반환하는 새로운 Promise를 만들 수 있습니다. 그리고 각각의 Promise의 데이터에 접근이 가능해집니다.

```javascript
const promiseOne = new Promise((resolve, reject) =>
                       setTimeout(resolve, 3000));
const promiseTwo = new Promise((resolve, reject) =>
                       setTimeout(reject, 3000));
Promise.allSettled([promiseOne, promiseTwo]).then(data => console.log(data));
Promise {<pending>}
//(2) [{…}, {…}]
//0: {status: "fulfilled", value: undefined}
//1: {status: "rejected", reason: undefined}
//length: 2
```

## String.prototype.matchAll

주어진 문자열과 정규표현식에 대해 `matchAll()` 메서드는 `capturing groups`를 포함, 정규표현식에 대응되는 모든 값의 iterator를 반환합니다.

구문: `str.matchAll(regexp)`

- regexp: 정규표현식
- 반환값: iterator

`/g` 와 함께 `match()`를 사용하면 capture group은 무시됩니다.

```javascript
const regexp = /g(ro)(up(\d?))/g;
const groups = 'group1group2group3';
groups.match(regexp);
//(3) ["group1", "group2", "group3"]
//0: "group1"
//1: "group2"
//2: "group3"
```

`matchAll`을 사용하면 capture group 접근이 가능합니다.

```javascript
const regexp = /g(ro)(up(\d?))/g;
const someString = 'group1group2group3';
const array = [...someString.matchAll(regexp)];
array;
//(3) [Array(4), Array(4), Array(4)]
//0: (4) ["group1", "ro", "up1", "1", index: 0, input: //"group1group2group3", groups: undefined]
//1: (4) ["group2", "ro", "up2", "2", index: 6, input: //"group1group2group3", groups: undefined]
//2: (4) ["group3", "ro", "up3", "3", index: 12, input: //"group1group2group3", groups: undefined]
//length: 3
```

## Optional Chaining Operator

long chain 방식으로 property에 접근하는 것은 에러를 발생시키기 쉽습니다. 또한 property의 존재를 확인해 가는 방식은 깊이 중첩된 구조를 만들어내기 쉽습니다.

예:

```javascript
let car = {
  engine: {
    consumption: 10
  }
};
```

consumption 값에 접근하려고 한다면,

```javascript
let comsumption = car.engine.consumption;
// console.log(consumption)
// 10 Ok, no problem here.
```

하지만 engine 이 null 값이라면? error일 것입니다. 그래서 property 체크부터 해야만 했습니다.

```javascript
let car = {};
car.engine.consumption;
//Uncaught TypeError: Cannot read property 'consumption' of
//         undefined
//Before ES2020
//Check if exists
let consumption = car.engine ? obj.engine.consumption : undefined;
//console.log(consumption )
//undefined
/////or/////
let car = {};
//Check if exists
let consumption;
if (car.engine && obj.engine.consumption) {
  let consumption = obj.engine.consumption;
} else {
  let consumption = undefined;
}
console.log(consumption);
//undefined
```

ES2020에서는 아래처럼 사용이 가능합니다.

```javascript
let car = {};

let consumption = car.engine?.consumption;
console.log(consumption);
//undefined
```

연속적인 형태도 가능합니다.

```javascript
let car = null;
let consumption = car?.engine?.consumption;
console.log(consumption);
```

Optional Chaining은 array에도 사용이 가능합니다.

```javascript
let array = null;

//This makes sure that array exists before trying to access the //first element.
let car1 = array?.[1];
```

Optional Chaining Operator는 단순하지만 매우 유용합니다.

## Dynamic Import

`Dynamic Import`는 function 형태의 lazy loading이 사용 가능한 새로운 static import 방법입니다.

`./greetingsModule.js`가 있다고 생각해 봅시다.

```javascript
//greetingsModule.js

export hello() => console.log("Hello World!");
```

`./greetingsModule.js` 모듈을 불러오는 방법은

```javascript
//main.js

import * as greet from './greetingsModule.js';

greet.hello();
// Hello World!
```

static import 구문은 반드시 파일의 최상단에 위치해야만 합니다.

static import 대신 Dynamic Import를 사용하면 promise 방식으로 코드의 어느 곳이든 필요할 때 호출이 가능해집니다.

```javascript
...
if( 1 === 1){
import('./greetingsModule.js').then( (greet) => {
             greet.hello();
            // Hello World!
         });
}
```

`async/await`도 사용 가능합니다.

```javascript
async function load() {
  let greet = await import('./greetingsModule.js');
  greet.hello();
  // Hello!
}
```

static, dynamic 방식 모두 유용하므로 필요한 곳에 적절하게 사용하는 것이 좋습니다.

## BigInt

Javascript의 숫자는 숫자 범위가 고정되어 있어 매우 큰수를 처리하는데 한계가 있습니다. 이러한 문제를 해결하기 위해 `BigInt`가 추가됩니다.

`BigInt`는 임의의 길이를 가지는 integer입니다. BigInt 타입의 변수는 253개의 숫자로 이루어질 수 있으며 2⁵³값으로 제한되지 않습니다.

```javascript
console.log(Number.MAX_SAFE_INTEGER);
//9007199254740991

const max = Number.MAX_SAFE_INTEGER;

console.log(max + 1);
//9007199254740992  -> Correct value!

console.log(max + 10);
//9007199254741000  -> Incorrect value! (1001)
```

이제 숫자 끝에 `n`을 추가함으로서 `BigInt`타입으로 정의 내릴 수 있습니다.

```javascript
const myBigNumber = 9007199254740991n;

console.log(myBigNumber +1n);
//9007199254740992n  -> Correct value!

console.log(myBigNumber +10n);
//9007199254741001n  -> Correct value!
//Note: 

console.log(myBigNumber +10); 
//Error: you cannot mix BigInt and other types, use explicit //conversions.

//Correct way: You have to add the letter 'n' on the end of the //number
```