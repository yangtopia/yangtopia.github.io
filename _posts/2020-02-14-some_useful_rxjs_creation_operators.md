---
layout: post
title: '유용한 Rxjs Creation Operators'
categories: [rxjs]
image: https://images.unsplash.com/photo-1521120413309-42e7eada0334?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
featured: true
hidden: false
---

> [원문: Some Useful Rxjs Creation Operators](https://images.unsplash.com/photo-1521120413309-42e7eada0334?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80)

Rxjs는 Reactive Programming을 하기 위한 라이브러리중 하나입니다. Creation Operator는 Observer를 통해 subscribe 되는 다양한 데이터를 Observable 로 만들어주는데 유용합니다.

## Ajax

API 호출에 대한 fetch response에 대해서는 `ajax()`를 사용할 수 있습니다.

```javascript
const observable = ajax(`https://api.github.com/meta`).pipe(
  map(response => {
    console.log(reponse);
    return response;
  }),
  catchError(error => {
    console.log(error);
    return of(error);
  })
);

observable.subscribe(res => console.log(res));
```

api response에 `map` operator를 이용한 `pipe`를 했습니다. 또한 `catchError` operator를 통해서는 HTTP error에 대한 예외처리도 할 수 있습니다.

그리고 `ajax.getJSON()`를 통해 더 단순화 할 수 있습니다.

```javascript
import { ajax } from 'rxjs/ajax';
import { map, catchError } from 'rxjs/operators';
import { of } from 'rxjs';
const observable = ajax.getJSON(`https://api.github.com/meta`).pipe(
  map(response => {
    console.log(response);
    return response;
  }),
  catchError(error => {
    console.log('error: ', error);
    return of(error);
  })
);
observable.subscribe(res => console.log(res));
```

두가지 샘플 코드 모두 `map` operator를 통해 `response`를 리턴합니다.

POST request에도 사용이 가능합니다.

```javascript
import { ajax } from 'rxjs/ajax';
import { map, catchError } from 'rxjs/operators';
import { of } from 'rxjs';

const observable = ajax({
  url: 'https://jsonplaceholder.typicode.com/posts',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: {
    id: 1,
    title: 'title',
    body: 'body',
    userId: 1
  }
}).pipe(
  map(response => console.log('response: ', response)),
  catchError(error => {
    console.log('error: ', error);
    return of(error);
  })
);

observable.subscribe(res => console.log(res));
```

보시는 것처럼 header와 body 값을 설정 가능하기 때문에 `ajax` operator를 통해 대부분의 HTTP 요청을 처리할 수 있습니다.

`catchError` operator를 통해서는 Error 처리도 가능합니다.

## bindCallback

`bindCallback`은 API callback을 Obsevable을 리턴하는 function으로 변환합니다.
parameter와 함께 정의된 function을 parameter를 방출하는 Observable로 변환할 수 있습니다.

`bindCallback`은 3가지 인수를 받을 수 있습니다.

첫번째는 parameter에 대한 callback function을 받는 인수입니다. callback function으로 무엇이 전달되든지 상관없이 Observable에 의해 emit됩니다.

~~두번째는 `resultSelector`이며 선택사항입니다. emit된 결과를 선택하는 함수를 전달할 수 있습니다.~~ **DEPRECATED**

마지막 인수는 `scheduler`입니다. 선택사항입니다. 첫번째 인수의 callback function이 호출되는 시점을 변경하려는 경우 scheduler를 전달할 수 있습니다.

```javascript
import { bindCallback } from 'rxjs';

const foo = fn => {
  fn('a', 'b', 'c');
};

const observableFn = bindCallback(foo);
observableFn().subscribe(res => console.log(res)); // ["a", "b", "c"]
```

`foo`의 parameter로서 `fn` callback function에 전달된 `a`, `b`, `c`가 출력되는 것을 확인 할 수 있습니다.

`bindCallback` function으로 Observable을 리턴하는 함수를 만들었습니다. 이제 리턴된 Obsevable를 subscribe 할 수 있습니다.

## defer

![defer](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/defer.png)

subscribe할 때만 만들어지는 Observable을 `defer`를 통해 만들 수 있습니다.

Obsevable factory function 형태의 단일 인수를 가집니다.

```javascript
import { defer, of } from 'rxjs';

const clicksOrInterval = defer(() => {
  return Math.random() > 0.5 ? of([1, 2, 3]) : of([4, 5, 6]);
});
clicksOrInterval.subscribe(x => console.log(x)); // [1, 2, 3] or [4, 5, 6]
```

`Math.random()`값에 따라서 `of([1, 2, 3])` 또는 `of([4, 5, 6])`를 subscribe하는 Observable을 만들 수 있습니다.

## empty

![empty](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/empty.png)

아무것도 emit하지 않는 Observable을 만듭니다.

한가지의 선택적 인수로서 scheduler를 사용할 수 있습니다.

```javascript
import { empty } from 'rxjs';

const result = empty();
result.subscribe(x => console.log(x)); // nothing logged
```

아무것도 출력되지 않습니다.

홀수 값을 출력하는 예를 들어봅시다.

```javascript
import { empty, interval, of } from 'rxjs';
import { mergeMap } from 'rxjs/operators';

const interval$ = interval(1000);
const result = interval$.pipe(
  mergeMap(x => (x % 2 === 1 ? of('odd') : empty()))
);
result.subscribe(x => console.log(x)); // odd, odd....
```

## from

![from](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/from.png)

`from`은 araay, array-like object, promise, iterable object, Observable-like object를 Observable로 만들어주는 operator입니다.

2개의 인수를 받으며 그중 선택적 인수로 scheduler를 받을 수 있습니다.

```javascript
import { from } from 'rxjs';

const array = [1, 2, 3];
const result = from(array);

result.subscribe(x => console.log(x));
```

promise를 Obsevable로 만들때도 사용합니다.

```javascript
import { from } from 'rxjs';

const promise = Promise.resolve(1);
const result = from(promise);

result.subscribe(x => console.log(x));
```

from을 사용하면 API 호출로 인한 promise를 Observable로 만드는데 매우 유용합니다.

지금까지 소개해 드린것처럼 다양한 형태의 데이터를 Observable 형태로 변환해 주기 때문에 creation operator는 매우 유용합니다.

`ajax`를 통해서는 HTTP 요청에대한 response를, `bindCallback`은 callback function의 인수를 Observable 데이터로 변환합니다.
또한 `defer`는 subscribe가 될때만 Observable이 생성될 수 있게 해줍니다.

`empty`는 아무것도 emit하지 않는 Observable, `from`은 array 또는 iterable한 object, Observable 형태의 object를 Observable 형태로 만들어 줍니다.
