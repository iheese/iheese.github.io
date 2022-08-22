---
layout: post
title: 'JPA BOARD TOY PROJECT 회고록'
subtitle: 'Spring boot, JPA, H2, Ajax, Spring Security, Test Code'
date: 2022-08-01 12:00:00 +0900
categories: 'PROJECT'
background: '/img/posts/project/project.jpg'
---

### JPA와 Spring Security를 이용한 게시판 프로젝트입니다.

- #### [ 어떤 프로젝트인지 자세한 내용 + 소스 코드 클릭!! ](https://github.com/iheese/JPABoardProject)

<br>

## 생각했던 점

#### RESTful, Ajax
- 처음으로 Ajax를 사용해보면서 REST한 요청을 신경써서 구현해볼 수 있었다. 
- 이번 프로젝트 요청 수는 작아서 괜찮았지만 정말 요청의 수도 늘고 복잡해지면 REST한 URI를 구현하는 것도 많은 고민이 필요할 것 같았다.
- 확실히 Controller 로직 구현시 REST한 요청으로 받으려 하니 코드가 직관적으로 잘 읽히고 보이는 점이 좋았다.

<br>

#### DTO(Data Trasfer Object)
- 사실 프로젝트를 모두 완성하고 제출 후에 팀원들과 회고 시간에 듣게 되어 코드를 수정했다.
- 실제로 프로젝트를 진행하면서 객체가 프론트 단으로 넘어가면서 필요하지 않은 속성값들도 같이 보내지는 것이 신경쓰였는데 DTO를 이용하여 구분하니 더 안정적으로 느껴졌다.
- Entity와 DTO로 구분하여 객체를 전달하는 것이 보안과 DB 영속성 유지 측면에서 좋다는 것을 알게 되었다.
- 덧붙여서 DTO와 Entity 변환 위치에 대한 많은 이야기가 있었고, DTO의 접근 범위에 관한 많은 개발자들 사이의 의견이 있는 것이 재미있었다.
> - 변환 과정에서 ModelMapper 라이브러리를 사용해볼 수 있었다. 
> - [DTO의 사용 범위에 대하여 _ Tecoble](https://tecoble.techcourse.co.kr/post/2021-04-25-dto-layer-scope/) 좋게 읽은 글이라 기록! 감사합니다.

<br>

#### 첫 코드 리뷰

- 전 프로젝트는 항상 완성하고 혼자 끝! 이었는데 이번에는 나의 멘토님에게 코드 리뷰를 받을 수 있었다. 
- 정말 꼼꼼하게 봐주셔서 감사했고 내 코드의 문제도 알 수 있었고 새로운 시각으로 나의 코드를 볼 수 있게 되었다. 

<br>

#### 코드 리뷰 정리
- 커밋을 나누는 연습, 줄 간격, 들여쓰기, 띄어쓰기 포맷을 맞추는 연습이 중요하니 연습하기
- 매직 넘버, 매직 스트링 지양하기
- 코드를 효율적으로 짤 수 있는 방법 많이 생각해보기
- 다른 사람 코드 많이 읽고 스타일 보면서 좋은 점 흡수하기
- 사용하지 않는 코드 지우고 주석된 코드는 이유 설명해놓기
- Controller, Service 단에서 처리될 로직 잘 구분하기
- Test code도 꼼꼼하게 확인하고 작성하기
- 예외 처리 적절하게 이용해보기
- Optional 적절한 상황에 이용하기

<br>

## 어려웠던 점

####  Test Code 작성하기

- Test Code를 처음으로 작성해보았다. 어떤 개발자 분의 인터뷰 영상에서 실제 로직 짜는 것보다 테스트 코드 짜는 시간이 더 많다는 것을 본 기억이 있는데 정말 그런 것 같았다. 처음이기도 했고 낯선 부분이 많아 메소드들과 친해지는 시간이 오래 걸렸다. 
- Controller, Service, Repository 각각의 단위 테스트를 작성하려 노력했다. 
- Repository 단위 테스트는 DB 처리와 관련된 부분이라 Mock(가짜 객체)을 사용하는게 의미가 없다고 생각했다. 그래서 비교적 쉬웠다.
- Service 단은 Mock을 이용하는 부분은 적응이 되니 괜찮았는데 DTO를 프로젝트에 적용하자 어려워졌다. 내 프로젝트에서 Entity와 DTO 변환이 Service에서 이루어지기 때문이었다.
- Controller 단위 테스트가 내게 제일 어려웠던 것 같다. 특히 Spring Security가 적용되면서 자동으로 filter가 적용되어 403 에러가 계속 뜨는 것을 해결하는 게 오래 걸렸다.(`@AutoConfigureMockMvc(addFilters = false)`으로 필터 적용을 막았다.)
> - 하지만 이 test가 의미가 있는지 많은 의문이 들었다. 실제 로직에서 사용된 커스텀한 Spring Security가 사용되지 않았기 때문이다. 
> - 나중에는 Spring Security 적용된 test code 작성하는 방법도 알아볼 필요가 있을 것 같다. 

<br>

- 실제 로직에서 객체를 반환하지 않으면 test code 구현하는 것이 조금 까다롭게 느껴졌다. 심지어 나는 실제 로직을 짤 때 객체를 무조건적으로 반환하는 게 싫어서 void 메소드를 꽤 사용했는데 이 부분 때문에 애를 먹었다.
> - `doNothing()` 을 이용하면 되었지만 test가 의미있게 느껴지진 않았다.
- any(객체.class)를 적극 이용했는데 코드 리뷰 때 질문을 받았다. 
> - 나는 given 단계에서 특정 값을 주었을 때 특정 값을 리턴해라(when ~ thenReturn)하는 것이 문제와 정답을 함께 주는 것처럼 느껴지고 현재 작성하는 단위 테스트에 집중하기 위해 사용했다. 더 test하는 느낌, 집중된 테스트라 생각했는데 그 방법은 아니었나보다. 
- 관련하여 사용한 예시에 대해 더 공부하여 적절할 때만 사용할 수 있게 해야할 것 같다. 

<br>

#### Spring Security
- 일단 앞에서도 말했듯이 Controller 단 단위 테스트에서 Spring Security가 합쳐지면서 어려워졌다.
- 일반적으로 커스텀한 Spring Security를 사용하는데 Spring Security가 제공하는 로직은 복잡했다. 관련하여 포스팅을 남기기도 했다.
> - [[JAVA, SPRINGBOOT] SPRING SECURITY 훑어보고 커스텀하기](https://iheese.github.io/java/2022/07/27/springsecurity/)

- 정말 제대로 알고 필요한 부분을 가져와 사용하려면 더 학습이 필요할 것 같다고 생각이 들었다. 
- 그래도 학습하고 사용하면 할수록 정말 좋은 라이브러리라는 생각이 많이 들었다.

<br>

## 느낀 점
- 이번 프로젝트는 주어진 요구 사항 뿐만 아니라 다른 부분들까지 구현하면서 많은 것을 배우고 학습할 수 있었다. 공부를 하면 할수록 부족함이 많이 느껴진다. 하나 하나 차근 차근 해나가자! 

<br>

## 추가 학습할 키워드
- Builder
- Optional
- Enum 타입
- Spring Security
- JPA, Query Method
- Test Code + 테스트 주도 개발_켄트 백

<br>

Reference:
- 백엔드 개발자 양성 과정 _ 패스트 캠퍼스, 채규태 강사님의 개인 프로젝트 회고록입니다. 