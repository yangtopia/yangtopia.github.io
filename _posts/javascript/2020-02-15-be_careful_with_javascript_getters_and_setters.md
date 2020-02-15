---
layout: post
title: 'Javascript에서 getter, setter 사용 시 주의해야할 점'
categories: [javascript]
image: https://images.unsplash.com/photo-1537884944318-390069bb8665?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1650&q=80
featured: true
hidden: false
---

>[원문: Be careful with JavaScript getters and setters](https://medium.com/javascript-in-plain-english/be-careful-with-javascript-getters-and-setters-32752bb2d496)

object에 대한 기본 `get`및 `set`작업은 값이 각각 새 속성으로 설정되거나 기존 속성에서 검색되는 방법을 제어합니다. 그러나 주의를 기울이지 않으면 눈에 띄지 않는 실수를 저지르기가 매우 쉽습니다.

## 사례1

`get` 메서드는 항상 같은 값을 반환합니다. `a`에 어떤 값을 부여하든 `foo.a`는 항상 1을 반환합니다.

```javascript
var foo = {
  get a() {
    return 1;
  }
};

foo.a = 5;
foo.b = 2;

const sum = (a, b) => a + b;

console.log(sum(foo.a, foo.b)); // 5 + 2 = 3 !!
```

## 사례2

이번 예시에서는 무한 재귀 호출을 발생시켜 브라우저가 다운됩니다.

`person.age` 속성을 참조하므로 JavaScript는 해당 속성을 가져와야합니다. 이 경우 `getter`가 트리거됩니다. JavaScript는 getter를 호출하지만 getter는 똑같은 작업(처리하려는 속성을 가져오라)을 수행하라는 지시를 받습니다. 이 함수는 항상 자체 호출하여 무한 재귀 호출을 만듭니다.

```javascript
function Person(age) {
  this.age = age;
}

Person.prototype = {
  get age() {
    return this.age; //Maximum call stack size exceeded!!
  }
};

var thePerson = new Person();

// This generates the error.
thePerson.age;
```

아래 예시의 `setter`에 대해서도 `getter`와 같은 이유로 에러가 발생됩니다.

```javascript
function Person(age) {
  this.age = age;
}

Person.prototype = {
  get age() {
    return this.age;
  },
  set age(age) {
    this.age = age; //Maximum call stack size exceeded!!
  }
};

// This generates the error.
var thePerson = new Person(40);
```

## 사례3

여기서는 반대로 값을 **다른 변수 `_a_`에** 할당합니다. `_a_`라는 변수의 이름은 특별한 의미나 규칙은 없습니다.
다른 속성과 마찬가지로 일반적인 속성입니다. **이렇게 하면 `get`과 `set`사용 시 재귀 호출을 피할 수**있습니다.

```javascript
var myObject = {
  get a() {
    return this._a_;
  },
  set a(val) {
    this._a_ = val;
  }
}

myObject.a = 5; // OK!!

console.log(myObject.a); // 5 OK!!
```

`getter`와 `setter`의 작동 방식을 이해하는데 도움이 되었기를 바랍니다.


