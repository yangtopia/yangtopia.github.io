---
layout: post
title: '자바스크립트 메모이제이션'
categories: [javascript]
image: assets/images/cover/photo-1519389950473-47ba0277781c.jpeg
featured: true
hidden: false
---

코딩을 하다보면 같은 값의 계산이 여러번 필요할 때가 있습니다. 한번 계산된 값을 저장하고 재사용하는 쉬운 방법은 뭘까요?

## 메모이제이션은 최적화 테크닉

메모이제이션은 연산결과를 캐싱 함으로서 함수의 성능을 끌어올리는데 초점이 맞춰져 있습니다. 계산했던 값이라면 재사용하고 그렇지 않으면 연산을 통해 새로운 결과값이 캐싱되는 원리입니다.

## 메모이제이션 없는 재귀 함수

팩토리얼을 계산하는 간단한 재귀함수를 봅시다.

```javascript
const factorial = num => {
  if (num < 0) {
    throw new Error('Number must be positive.');
  }
  if (num === 0 || num === 1) {
    return 1;
  }
  return num * factorial(num - 1);
};
factorial(3); // 6
factorial(4); // 24
```

3팩토리얼은 6, 4팩토리얼은 24임을 계산했습니다. 함수를 보면 팩토리얼값을 계산하는 함수가 변수에만 근거해 연산되는 순수 함수임을 알 수 있습니다. 그렇다면 생각해봅시다. 아주 큰 수의 팩토리얼 값을 여러번 계산해야 하는 경우에 성능이 저하될까요? 한번 계산 됐던 입력에 대한 결과를 저장할 수 있으면 어떨까요? 외부 데이터베이스 없이 어떻게 구현할 수 있을까요?

## Higher Order Function을 활용한 메모이제이션 유틸 함수

결과값을 캐싱할 메모이제이션 함수를 작성해 봅시다.

```javascript
const memoizeUtil = func => {
  const cache = {};

  return input => {
    return cache[input] || (cache[input] = func(input));
  };
};
```

코드에서 모든 결과값은 cache object에 저장됩니다. 데이터베이스 역할인 셈이죠.

클로저 형태의 `memoizeUtil` 함수는 둘 중 하나로 동작합니다.

1. cache에 입력 인수에 대한 값이 있으면 그에 대한 값을 반환
2. 그렇지 않으면 전달 된 함수를 이용하여 입력을 계산하고 cache 업데이트

## 메모이제이션 기능의 재귀 함수

이제 `memoizeUtil` 함수로 팩토리얼 함수를 랩핑해서 결과값을 캐싱할 수 있습니다.

```javascript
const findFactorial = memoizeUtil(num => {
  if (num < 0) {
    throw new Error('Number must be positive.');
  }
  if (num === 0 || num === 1) {
    return 1;
  }
  return num * findFactorial(num - 1);
});
```

좀더 체계적으로 코드를 다듬어 봅시다.

```javascript
const findFactorial = memoizeUtil(factorial);
function factorial(num) {
  if (num < 0) {
    throw new Error('Number must be positive.');
  }
  if (num === 0 || num === 1) {
    return 1;
  }
  return num * findFactorial(num - 1);
}
```

함수를 실행해 캐시된 값을 살펴봅시다.

```javascript
findFactorial(2); // 2
findFactorial(3); // 6
findFactorial(6); // 720
// 현재 저장된 캐시값:
// {2: 2, 3: 6, 6: 720}
findFactorial(6);
// 캐시에 저장된 값을 반환: 720
findFactorial(5); // 120
// 현재 저장된 캐시값:
// {2: 2, 3: 6, 5: 120, 6: 720}
```

## 메모이제이션 성능 테스트

`console.time()`와 `console.timeEnd()`로 성능 테스트를 해봅시다.

```javascript
console.time('factorial test no memo');
findFactorial(6); // 720
console.timeEnd('factorial test no memo');
// factorial test no memo: 0.110ms

console.time('factorial test with memo');
findFactorial(6); // 720
console.timeEnd('factorial test with memo');
// factorial test with memo: 0.019ms
```

성능이 좋아진 것이 보이죠? 더 큰 수를 넣어서 테스트해 봅시다.

```javascript
console.time('factorial test no memo');
findFactorial(100); // 9.33262154439441e+157
console.timeEnd('factorial test no memo');
// factorial test no memo: 0.264ms

console.time('factorial test with memo');
findFactorial(100); // 9.33262154439441e+157
console.timeEnd('factorial test with memo');
// factorial test with memo: 0.021ms
```

연산의 대상이 되는 숫자가 클수록, 재귀 호출 스택이 클수록 성능에 미치는 영향이 더 크다는 것을 알 수 있습니다. 컴퓨터의 성능에 따라 다소 차이는 있을 수 있지만 성능 향상이 된다는 점은 여전히 유효할 겁니다.

## console.log 테스트
**fincFactorial(100)**을 **console.log** 출력하는 테스트에서 메모이제이션 없이는 평균 2~4초 걸렸지만 메모이제이션을 구현하면 평균 100~300밀리초가 걸렸습니다.

## 결론
메모이제이션을 구현한 후에도 재귀 함수 자체의 변경사항은 크지 않았고 여전히 똑같이 동작합니다. 이러한 메모이제이션의 구현은 사이드이펙트가 없는 순수 함수에만 적용해야 합니다. 그래야 결과를 예측하기가 쉽습니다. 결과값을 일종의 룩업 테이블 형태로 메모리에 저장 후 엑세스 하는 형태 이므로 부하가 심하게 걸리는 연산일수록 그 장점이 두드러집니다.
