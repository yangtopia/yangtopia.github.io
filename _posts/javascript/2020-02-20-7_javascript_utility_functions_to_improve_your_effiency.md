---
layout: post
title: '작업 능률을 올려주는 7가지 Javascript 유틸리티 함수'
subtitle: 'Detect browser, detect function type, convert hyphen-case to camelCase, delete HTML tag in a string and reverse a string, etc.'
categories: [javascript]
image: https://images.unsplash.com/photo-1550439062-609e1531270e?ixlib=rb-1.2.1&q=85&fm=jpg&crop=entropy&cs=srgb
featured: true
hidden: false
---

> [원문: 7 Javascript Utility Functions to Improve Your Effiency](https://medium.com/javascript-in-plain-english/7-javascript-utility-functions-to-improve-your-efficiency-79d27132f186)

## Detect Browser: 브라우저 유형 확인

각각의 브라우저는 서로 다른 `navigator.useragent`값을 가지고 있습니다. 이 값을 조회해서 어떤 브라우저인지 알아낼 수 있습니다.

예시(크롬 브라우저):

![chrome](/assets/images/posts/20200218/screenshot-1.png)

`navigator.useragent`에 있는 브라우저 브랜드 이름을 확인하면 브라우저의 유형을 확인할 수 있습니다.

```javascript
const inBrowser = typeof window !== 'undefined';

// get user agent
const UA = inBrowser && window.navigator.userAgent.toLowerCase();

// detect browser
const isIE = UA && /msie|trident/.test(UA);
const isIE9 = UA && UA.indexOf('msie 9.0') > 0;
const isEdge = UA && UA.indexOf('edge/') > 0;
const isChrome = UA && /chrome\/\d+/.test(UA) && !isEdge;
const isPhantomJS = UA && /phantomjs/.test(UA);
const isFF = UA && UA.match(/firefox\/(\d+)/);

// detect OS
const isAndroid = UA && UA.indexOf('android') > 0;
const isIOS = UA && /iphone|ipad|ipod|ios/.test(UA);
```

---

## Detect Function Type: 함수 유형 확인

여기 2가지 유형의 함수가 있습니다.

- runtime 환경에서 제공되는 native 함수들  
  `Array.isArray`, `console.log`.

- 개발자에 의해 작성된 함수들

다양한 경우에 있어 위 두가지 타입의 함수를 구분해야 할 필요가 있습니다.

그렇다면 어떻게 구분하는 것이 좋을까요? 아주 간단합니다. string으로 변환했을 때의 결과가 다릅니다.

```console
// Native Functions
> console.log.toString()
"function log() { [native code] }"

> Array.isArray.toString()
"function isArray() { [native code] }"

// user-written function
> function sayHi() { console.log('hi) }
undefined

> sayHi.toString()
"function sayHi() { console.log('hi) }"
```

native 함수를 string으로 변환하면 그 결과엔 항상 `native code`가 포함됩니다.

따라서 native 함수를 판별하는 함수를 작성한다면:

```javascript
function isNative(func) {
  return typeof func === 'function' && /native code/.test(func.toString());
}
```

---

## Convert hyp-hens to camelCase: hyp-hen 형태를 camelCase로 변환

`hello-world` string을 `helloWorld`스타일로 변환이 필요한 경우가 자주 있습니다. 이를 위해서는 정규식을 사용할 수 있습니다.

`/-(\w)g`정규식을 `-`이후의 소문자를 찾고 대문자로 변경하는데 사용할 수 있습니다.

```javascript
function camelize(str) {
  const camelizeRE = /-(\w)/g;
  return str.replace(camelizeRE, (_, c) => (c ? c.toUpperCase() : ''));
}

camelize('hello-world'); // "helloWorld"
camelize('A-p-p-l-e'); // "APPLE"
```

---

## Delete HTML tag in string: string에서 HTML tag 제거

보안 문제로 string에서 HTML tag를 제거해야 할 경우가 자주 있습니다. 간단한 정규식으로 쉽게 처리가 가능합니다.

```javascript
const stripHTMLTags = str => str.replace(/<[^>]*>/g, '');
stripHTMLTags('<p><em>Hello</em> <strong>World</strong></p>'); // "Hello World"
```

---

## Reverse a String: string 뒤집기

string을 배열로 나눈후에 뒤집은 뒤에 다시 합치면 됩니다.

```javascript
const reverseStr = str =>
  str
    .split('')
    .reverse()
    .join('');
reverseStr('hello wolrd'); // "dlrow olleh"
```

---

## Format a number to a string with comma: 숫자에 콤마(,)표시 추가하기

큰 숫자를 읽기 쉽게 표현하기 위해서는 콤마(,)를 이용합니다.

- `111111` => `111,111`
- `12345678` => `123,456,789`

일반적으로 3번째 숫자마다 콤마를 추가합니다.

```javascript
function numberToStringWithComma(number) {
  // convert number to string
  let str = String(number);

  let s = '';
  let count = 0;
  for (let i = str.length - 1; i >= 0; i--) {
    s = str[i] + s;
    count++;
    // add a comma to every three numbers
    if (count % 3 == 0 && i != 0) {
      s = ',' + s;
    }
  }
  return s;
}
numberToStringWithComma(11111111); // "11,111,111"
numberToStringWithComma(111111112342342340); // "111,111,112,342,342,340"
```

---

## Convert byte units to reasonable units: byte단위의 숫자를 읽기 쉽게 변환하기

컴퓨터 파일의 경우 bytes 단위로 표현되는 것이 일반적입니다. 하지만 아주 큰 숫자일 경우 사람이 읽기 힘듭니다.

예를 들면, 98223445B의 경우 직관적으로 얼마나 큰지 알기가 어렵죠. 하지만 93.7MB로 표현한다면 읽기 쉬울 겁니다.

```javascript
function byteToSize(bytes) {
  if (bytes === 0) return '0 B';

  var k = 1024;
  var sizes = ['B', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'];
  var i = Math.floor(Math.log(bytes) / Math.log(k));
  return (bytes / Math.pow(k, i)).toPrecision(3) + ' ' + sizes[i];
}

bytesToSize(12); // "12.0 B"
bytesToSize(234234); // "229 KB"
bytesToSize(98223445); // " 93.7 MB"
bytesToSize(3498223445); // "3.26 GB"
bytesToSize(3436498223445); // "3.13 TB"
bytesToSize(6853436398223445); // "6.09 PB"
```
