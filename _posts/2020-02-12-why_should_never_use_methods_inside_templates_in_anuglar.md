---
layout: post
title: 'Angualr 템플릿안에서 메서드를 절대 사용해서는 안되는 이유'
categories: [angular]
image: https://images.unsplash.com/photo-1550439062-609e1531270e?ixlib=rb-1.2.1&q=85&fm=jpg&crop=entropy&cs=srgb
featured: true
hidden: false
---

> [원문: Why You Should Never Use Methods Inside Templates in Angular](https://medium.com/better-programming/why-you-should-never-use-methods-inside-templates-in-angular-497e0e11f948)

앵귤러는 웹 어플리케이션을 쉽게 만들 수 있도록 도와주는 훌륭한 프레임워크중 하나입니다. 앵귤러의 핵심 기능 중 하나는 새로운 DOCTYPE 선언 없이 HTML 템플릿을 사용하는 것에 있습니다. 앵귤러에서는 개발자가 원하는 HTML 태그를 자유롭게 사용할 수 있는데 그 이유는 모든 HTML 태그들이 DOCTYPE, metadata, script와 style link를 포함하고 있는 `index.html` 파일에 포함되기 때문입니다.

앵귤러의 템플릿 파일들이 DOCTYPE 과 metadata를 필요로 하지 않는다는 것이 핵심은 아닙니다. 핵심은 일반 HTML 파일에서는 불가능한 일을 할 수 있다는 것에 있습니다. 그중 한가지는 어디서나 double brace를 사용할 수 있다는 것이고 그 안에 TypeScript를 포함할 수 있다는 것입니다.

```html
<div *ngIf="currentUser$ | async as user">
  <img
    alt="profile_pic"
    height="150px"
    src="{% raw %}{{user.image}}{% endraw %}"
    width="150px"
  />
</div>
```

`user` object에서 `image`값을 `img`태그의 `src`속성에 할당했습니다. 꽤 유용한 기능입니다. `user` 변수를 사용한것 처럼 메서드도 double brace안에서 사용이 가능합니다.

```html
<textarea
  value="{% raw %}{{getAddress()}}{% endraw %}"
  placeholder="Address"
></textarea>
```

user object에서 주소에 대한 세부 값을 참조하고 문자열을 연결해 반환하는 `getAddress()` 메서드를 호출했습니다. `getAddress()`메서드는 아래처럼 생겼습니다.

```javascript
getAddress() {
  let address = '';
  address += `${this.user.address.address_line}\n`;
  address += `${this.user.address.city}, `;
  address += `${this.user.address.state} `;
  address += `${this.user.address.zip}`;

  return address;
}
```

잘못될게 없겠죠? 제대로 주소가 출력될겁니다. 하지만 뭔가 잘못됐습니다.

무엇이 잘못됐는지 알아보기 위해 메서드의 실행 시작지점에 `console.log`를 넣어봅시다.

```javascript
getAddress() {
  console.log('Entered getAddress()');
  let address = '';
  address += `${this.user.address.address_line}\n`;
  address += `${this.user.address.city}, `;
  address += `${this.user.address.state} `;
  address += `${this.user.address.zip}`;

  return address;
}
```

이제 메서드가 호출될때를 알 수 있게 됐습니다. 당연히 단 한번 콘솔이 출력될것으로 생각될겁니다. 하지만 무슨일이 벌어질까요?

콘솔을 열고 페이지를 새로고침하면!

![1.png](/assets/images/posts/20200212/1.png)

`getAddress()`가 새로고침 후 4번이나 호출됐습니다. 그리고 페이지를 클릭하거나 textarea에 마우스를 올리거나 할때 계속 콘솔이 출력됩니다.

이러한 현상은 Angular의 change detection 때문입니다. 정확히 말하면 change detection의 문제가 아니고 우리 코드의 문제죠. 템플릿에서 double brace안에 메서드를 사용하는건 좋지 않습니다. 한번의 호출로 address값을 할당하는 것이 가장 좋습니다.

```javascript
ngOnInit() {
  this.address = this.getAddress();
}
```

```html
<textarea
  value="{% raw %}{{address}}{% endraw %}"
  placeholder="Address"
></textarea>
```

이렇게 하면 `getAddress()` 메서드는 한번만 호출됩니다. `address` 변수에 user의 주소 문자열 조합도 할당됩니다.

콘솔을 통해 확인해 보면 한번만 찍히는걸 확인할 수 있습니다.

![2.png](/assets/images/posts/20200212/2.png)

`getAddress()`가 불필요하게 여러번 호출되면 우리의 앱은 느려질 겁니다.

이러한 단순한 경우로 그런일은 발생하지 않겠지만 많이 쌓이다 보면 성능에 큰 영향을 미칠겁니다.

***

## 결론

Angular 프레임워크를 사용하면 기존의 HTML이 제공하지 않는 많은 기능을 사용할 수 있습니다.

하지만 이러한 기능을 생각없이 쓰다보면 Angular를 안쓰는게 나을지도 모릅니다. 사실상 대부분의 문제는 우리가 짜놓은 코드에서 발생됩니다. 우리가 문제를 알아차리지 못할 뿐이죠.

읽어주셔서 감사합니다!