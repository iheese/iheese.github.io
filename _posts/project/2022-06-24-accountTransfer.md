---
layout: post
title: 'ACCOUNT TRANSFER TOY PROJECT with SPRING, MYBATIS  회고록 '
subtitle: 'Spring, MVC, H2, MyBatis, Transaction'
date: 2022-06-27 12:00:00 +0900
categories: [project]
background: '/img/posts/project/project.jpg'
---

### 프로젝트 설명
- Spring과 MyBatis를 이용한 계좌 이체 프로젝트입니다.
- 개인 프로젝트이며, 계좌 이체 로직 후 확인 페이지를 구현했습니다.
- 추가로  AOP를 이용하여 Transaction 로직을 구현해보았습니다.  
- #### [ 어떤 프로젝트인지 자세한 내용 + 소스 코드 클릭!! ](https://github.com/iheese/AccountTransferToyProject)

<br>

## 생각해봐야할 점
- 제가 작성한 코드와 강사님의 코드를 비교하며 회고한 내용입니다.


#### Dispatcher Controller가 데이터를 리턴할 때 꼭 세션을 이용해서 전달해야 했을까?
- 필자 생각에는  WOORI 은행 계좌가 계좌 조회 페이지, 이체 정보를 입력하는 사이트에도 필요하니 매번 model 객체에 담아서 전달하는 것이 코드의 중복을 일으킬 수 있겠다고 생각했다.
- 세션에 WOORI 은행 객체를 담아두고 브라우저가 꺼지지 않는 이상 유지될 수 있게 하여 사용하는 것은 괜찮은 생각이었던 것 같다. 

<br>
#### VO 객체를 하나로 하는 것은 어땠을까?
- 필자는 이체하는 계좌, 이체 받는 계좌 당연히 따로 만들었는데 강사님은 하나의 bank 객체를 만들어서 WOORI 계좌로 사용할 때도 있고, KBSTAR 계좌로 사용하시기도 하였다. 
- 이번 프로젝트의 경우 WOORI, KBSTAR의 객체 정보를 한 페이지 안에서 한번에 받는 경우가 없어서 bank 객체 하나로 공유하는 것이 가능하지 않았을까? 싶다. 
- 메모리를 아끼고 코드의 중복을 줄일 수 있다는 점은 좋은 것 같다.
- 로직을 처리해야 할 은행의 수가 많아진다면 강사님의 방법이 더 효율적인 처리가 될 것 같다.

<br>
#### 로직 처리를 어디서 진행해야할까?
- 필자는 과제를 받고 중요한 로직을 결제 로직이라고 생각했다. 그래서 돈을 보낸 계좌에서는 돈이 빠지고, 돈을 받은 계좌에서 돈이 더해지는 로직은 서비스 단에서 처리했다. DB 단에서는 뺀 돈을 update 하는 로직만 구성했다.
> - 필자처럼 구성하면 DispatcherServlet의 코드가 번잡해지긴 하는 것 같다.
> - 그래도 역할 분배 입장에서 보면 로직의 부담이 잘 나눠진 것 같다.
- 강사님의 코드에서는 sql문으로 돈을 빼고 더하는 로직이 처리되었고 서비스 단에서는 호출만 하여 돈이 빠지고 더해지는 로직이 순서대로 실행되었다. 즉 DB 처리 단에서 중요 로직이 처리되었다고 볼 수 있겠다.
> - 강사님의 방식은 은행 업무처럼 동시성 이슈가 민감하고, 로직 자체가 복잡하지 않은 분야에서는 좋은 코드라는 생각이 들었다.

<br>

## 어려웠던 점 및 느낀 점
- 로직 처리가 어디까지 서비스 단에서 처리되어야 하는지 정해야 하는 것이 어려웠다.
- JSP 파일에서 DispatcherServlet으로 데이터를 받아오고 처리 과정이 헷갈려서 어려움을 겪었다. 
- MyBatis에 대한 이해와 학습이 더 필요할 것 같다. 

<br>

Reference:
- 백엔드 개발자 과정 _ 패스트 캠퍼스, 채규태 강사님 강의의 개인 프로젝트 결과물 입니다.

  


