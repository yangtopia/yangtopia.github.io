---
layout: post
title: 'SPA는 어떻게 웹 설계를 변화시켰을까'
categories: [javascript, spa, web]
image: https://images.unsplash.com/photo-1513082325166-c105b20374bb?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
featured: true
hidden: false
---

>[원문: How Single Page Applications Broke Web Design](https://medium.com/javascript-in-plain-english/how-single-page-applications-broke-the-web-design-bd18d4ddcdab)

초창기의 웹은 하이퍼링크와 함께 결합된 문서의 개념으로 디자인 되었습니다. 웹 브라우저를 통해 서버에 요청을 보내면 컨텐츠를 다운로드 받고 그 내용(CSS로 꾸며진)을 화면에 표시해 주는 것이었죠. 하이퍼링크가 있는 문서는 다른 컨텐츠를 가진 새로운 문서로 이동이 가능하게 했고 이것들이 모여 웹을 구성합니다. 웹 브러우저, 크롤러, 소셜 미디어와 같은 모든 웹 기반 시스템들은 이와같은 단순한 아이디어에서 출발했습니다.

---

## 단일 페이지 어플리케이션 (Single Page Applications)
단일 페이지 앱(Single Page Application)는 Javascript의 도움으로 웹 페이지의 내용을 재작성하는 방식으로 사용자와 상호작용을 합니다. 사용자와의 모든 상호작용 마다 새로운 페이지를 불러오는 것 대신에, SPA는 외부 API를 통해 필요한 데이터를 다운로드 받고 불러오는 시간을 감소시켜 사용자 경혐을 향상시켜 줍니다. 이러한 앱들은 웹사이트와 달리 보통의 데스크탑 앱처럼 작동하는 것처럼 보이게 하므로 사용자에게 좀더 자연스러운 느낌을 줍니다.

`single page application`이란 용어는 2002년 처음 사용됩니다. 3년후, AJAX의 등장은 Ember나 Angular와 같은 SPA 프레임워크가 개발될 수 있는 기반을 제공해 주었습니다. 각각의 프레임워크는 문제를 해결하기 위한 서로 다른 목표를 가지고 있었습니다. Angular가 이미 HTML에 익숙한 웹 디자이너들을 목표로 개발되었던 것에 반해 Ember는 `convention over configuration`의 원리 아래 boilerplate 코드를 줄였습니다. 2011년 소개된 ReactJS는 코드 재사용, 속도 및 접근방식에 근거한 분리된 웹 컴포넌트를 중심으로 개발되었습니다.

## 시대에 뒤처진 기반 기술 (The underlying technology is outdated)
React와 같은 프레임워크는 개발자들이 전통적인 접근방식으로는 불가능한 복잡한 사용자 인터페이스를 쉽게 만들 수 있게 해주었습니다. 하지만 이러한 프레임워크가 구동되는 브라우저 환경의 일부는 정적인 웹사이트를 염두에 두고 설계되었죠.

예를 들면, URL이라는 개념은 웹사이트에 개념에는 부합하지만, SPA의 관점에서는 다시 생각해볼 필요가 있습니다. URL은 종종 브라우저의 History API에 의해 인위적으로 주입되는데 장기적 관점의 해결책이 아닌 절충적인 해결법으로 보입니다. 주의 깊게 신경쓰지 않으면 몇몇 SPA는 뒤로가기, 앞으로가기 같은 브라우저의 단순한 기능에도 문제를 일으켜 사용자 경험을 해치게 될 수도 있습니다. SPA에서는 title 요소같은 매우 단순것을 처리하는 것도 Javascript를 통해 특별한 방법으로 처리하거나 서버 사이드 렌더링으로 해결할 필요가 있습니다.

또다른 이슈는 웹 크롤러입니다. 구글 직원인 John Mueller에 따르면 구글은 이미 SPA구축된 웹앱에 대한 인덱스 작업을 진행하고 있지만 개선의 여지가 여전히 있다고 합니다.

> 항상 완벽하고 쉽지는 않지만 몇몇 사이트에 대한 작업은 잘 이루어지고 있습니다. 심지어 클라이언트 사이드 렌더링 앱에서도요. -John Muller

하지만 페이스북이 ReactJS를 개발한 뒤로 그들 스스로의 크롤러에서 조차 그들의 플랫폼(서버 사이드 렌더링 코드로 쌓아 올린 앱들)에서 Javascript의 *let*과 *var*의 혼합물로 만들어진 SPA에 대해서는 완전히 장님 수준이었습니다.

## 상태 관리의 어려움 (State management is tricky)

단일 페이지의 접근 방식은 유익하지만 한가지 명심해야 될 것이 있습니다. 전통적인 stateless 접근법과는 다른 application state 유지 방식에 따르는 어려움입니다. 아래의 시나리오로 디버깅하기 쉽고 가장 단순한 형태의 문제를 구현해보겠습니다.

- 사용자(리처드)는 당신이 이제 막 만들어낸 React SPA에 방문합니다. 그리고 당신이 몇달 동안 땀흘려 만든 탭 스위치의 빠른 동작에 진심으로 감탄합니다.
- 리처드는 회원가입 페이지로 이동하고 빠르게 필요한 정보를 입력해 당신의 앱에 가입하려 합니다.
- 회원이 된 후 리처드는 **스토리**와 **읽기 목록**이라는 새로운 탭 2개를 볼 수 있게 되었습니다. 리처드는 **스토리**탭을 열고 그의 읽기 목록에 빠르게 두개의 스토리를 추가합니다.
- 하지만 읽기 목록에는 여전히 **0 스토리**로 표시되는 것에 혼란을 느낀 리처드는 그의 계정을 삭제해 버립니다.

![image](https://miro.medium.com/max/660/0*tRA4zIPhfibeE7jw.png)

`과거`의 기술로는 두개의 서로 다른 문서, stories.php, reading-list.php로 구현 되기 쉽습니다. 각각 명확하게 정의된 라이프사이클을 가진 독립된 프로그램으로서 말입니다. -- 리처드가 브라우저에서 링크를 클릭하면 프로그램은 몇초동안 실행된 후 페이스북 크롤러가 읽기 쉬운 형태의 HTML 소스 코드로 응답을 보냈을 것입니다.

SPA의 경우 app.js라는 오직 하나의 프로그램만이 존재하는 경우가 대부분입니다. 이것은 실행되기 전 리처드의 브라우저에 전송되며 브라우저는 논리적 해석과 연산 후에 DOM 요소들로 변환합니다.

리처드로 돌아옵시다. 그가 경험한 문제는 부적절한 상태관리에서 비롯된 것입니다. 프로그래머는 사용자가 스토리를 읽기 목록에 추가하는 버튼을 눌렀을 때 읽기 목록에 보여야 한다는 상태를 업데이트 해야했지만 목록을 업데이트 하는 것을 잊어버리고 말았습니다. 이러한 문제는 쉽게 해결될 수 있지만 앱의 크기가 커지면 커질수록 해결하기 어려워 질 수도 있습니다. Javascript 커뮤니티에서 상태관리를 다루는 것에 많은 도움을 얻을 수 있고 대부분의 모바일 개발자들도 이러한 문제에 익숙합니다. Redux와 같은 툴을 사용하면 골치아픈 상황을 조금은 해결할 수 있지만 boilerplate와 프로젝트의 복잡도가 증가하는 경향이 있는것이 문제입니다.

---

웹 개발자들은 빠르게 새로운 기술로 이동하고 있습니다. SPA는 전통적인 방식과 비교했을 때 화면의 전환과 같은 것에서 놀라운 사용자 경험을 제공하고 있습니다. 지엽적인 문제를 해결하기 위해 매일 같이 새로운 라이브러리와 프레임워크가 릴리즈되고 있습니다. 앞으로의 과제는 브라우저들과 World Wide Web이 SPA를 잘 실행시키도록 하는 것일 겁니다. 그러기 전까지는 다음 **"big thing"**에 대한 얘기는 무의미한 것일 겁니다.