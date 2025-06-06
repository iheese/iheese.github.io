---
layout: post
title: 'Finance Product Recommendation Mini Project  회고록 '
subtitle: 'Spring boot, JPA, MySQL, Docker, AWS, 협업'
date: 2022-09-08 12:00:00 +0900
categories: [project]
background: '/img/posts/project/project.jpg'
---

### 프로젝트 설명

- 프론트엔드팀과 백엔드팀이 협업한 대출 상품 사이트 미니 프로젝트입니다.
- 나이와 직업을 바탕으로 대출 상품을 추천해주는 프로그램입니다.  
- #### [ 어떤 프로젝트인지 자세한 내용 + 소스 코드 클릭!! ](https://github.com/iheese/FinProductReco)

<br>

## 어려웠던 점, 아쉬웠던 점

### 협업
- 나의 첫 팀 프로젝트는 나의 부족함을 몸소 뼈 저리게 느낄 수 있었다.
- 팀원 분의 추천으로 백엔드팀의 팀장을 하게 되었는데 수많은 미팅과 세세하게 정해지는 부분을 많이 놓쳤던 것이 아쉽다.
- 백엔드 우리 팀원들과 의사소통, 프론트엔드팀과의 의사소통, 모든 것이 어려웠고 부족했던 것 같다. 
- 남의 코드를 이해하고 나의 코드와 섞어서 하나의 결과물을 내는 것은 정말 어려운 일인 것 같다. 누가 봐도 이해하기 좋은 코드를 짜기 위해 더욱 노력해야겠다. 
- 또한 우리끼리 자주 모여서 코드 리뷰를 했으면 좋지 않았을까? 좋은 코드를 내기 위해 더 많은 소통이 필요했음을 끝나서야 알게 되었다.  

<br>

#### HTTPS
- 마지막까지 실패했던 부분이었다. 
> - 평소에 당연하게 여기고 사용했던 것이 그 과정은 정말 복잡했다. 
- 배포를 앞두고 HTTPS 설정에 굉장히 많은 시간을 투자했으나 쉽지 않았다.
> - 사실 멘토님의 조언으로 Nginx 걷어내고 배포를 시도했으나 프론트 측에서 Netilfy를 사용하는데 HTTPS 설정이 필요하다고 요청하셔서 Nginx  웹 서버에 let's encrypt를 통해서 HTTPS 설정이 잘되지 않았다. 
- 막판에 알게 된 사실, 스프링 부트 프로젝트 자제에도 HTTPS 설정을 할 수 있다고 한다. 

<br>

#### AWS, 배포
- 간간히 특강을 통해 배포에 대한 호기심 반, 걱정 반이 있었는데 이번에 AWS를 이용한 배포는 처음 시도해보았다.
- 일단 신기하고 재밌었고 학습할 부분이 정말 많은 것 같았다. 
> - 몇 가지 설정을 통해 개발자가 신경써야 할 부분이 모두 처리되는 것이 신기했고 그 과정에 대해 더 학습할 필요성을 많이 느꼈다. 

<br>

#### Docker
- 도커는 멘토님의 도움이 매우 컸다. (나에게 미지의 대상이었다.)
- Portainer를 이용해 도커를 관리하고 MySQL 이미지를 띄어 같은 환경에서 함께 사용할 수 있게 했다.  
- 우리 프로젝트에서는 DB 정도만 올려 사용했지만 더 많은 사람들이 참여하고, 많은 컨테이너가 필요한 프로젝트라면 그 영향력이 엄청날 것 같다고 생각했다. 

<br>

#### CI / CD
- 배포하는 과정을 따라가는 것도 힘들어서 이 부분은 거의 진행하지 못했다. 
- EC2 서버를 띄우고 Spring Boot 프로젝트를 빌드해 CyberDuck이라는 프로그램으로 로컬에서 서버로 jar 파일을 올려 진행했다. (프리티어를 사용하고 있어 이 방법이 안전하다고 하기도 하셨다.)
> - git을 깔아서 pull 하는  방식, scp를 이용해 로컬에서 ec2로 편하게  전송할 수도 있다고 한다. 
> - 이 반복적인 과정을 거치면서 이래서 CI/CD가 필요함을 몸소 알았다. 다음 프로젝트는 꼭 시도해볼 것이다.

<br>

#### 테이블 매핑
- 테이블(User, Cart, Product)이 3개 밖에 없는 매핑이었는데 초기에 생각을 잘못해서 설계를 잘못했다. 
- 프로젝트 막판에 Cart와 Product가 원하는 대로 작동하지 않아 결국 ManyToMany로 변경했다. 결국 작동하긴 했지만 현업에서 사용하지 않는 이유가 두 엔티티 사이에 생기는 조인테이블로 인한 예상치 못한 쿼리 발생으로 성능이 좋지 않아진다고 한다. 또한 조인 테이블에 생길 수 있는 필드가 있을 수 있다고 한다. 
- 다대다 말고 일대다, 다대일로 연결테이블을 엔티티로 만들어서 처리해주는 것이 일반적이라고 한다.


![](/img/posts/fun/아우 되게 어렵네.jpg){: width="550"}


<br>

## 신경썼던 점

#### JWT 토큰
- 이번 프로젝트 백엔드팀의 주된 과제 중 하나였다. 
- 처음에는 Spring Security와 JWT를 이용해서 구현하려 했으나 멘토님, 팀원들과 회의 후에 JWT 토큰과 Interceptor를 이용하여 구현하였다.
- 배포하고 서버에서 프론트팀에게 로그인시 토큰을 넘겨 주고 토큰을 이용해 추가적인 데이터를 요청할 때 짜잘한 문제들이 생겨 아쉬웠다.
> - 개인적인 생각으로 토큰 방식이 정말 마냥 좋다고는 못할 것 같다.
- 다음에 구현한다면 Payload에 들어갈 데이터를 구제적으로 보안적인 부분까지 고려해 설계할 것 같다.
- Redis의 블랙리스트를 구현하여 로그아웃한 토큰을 저장하는 방식도 시도해보면 좋을 것 같다.

`+` 추가적으로 멘토님께서 해주신 말 : 유저 엔티티에는 보안 때문에 절대 password가 있으면 안된다! (공부해볼 여지가 있다. )

<br>

#### DB
- 왜 나는 H2만 생각했을까? 당연히 무난하게 서버에 H2 DB 올려서 인메모리로 할 수 있지 않을까? 라는 막연한 생각이 있었다. 
- [Deploy a Spring Boot App with H2 Database on AWS](https://stackoverflow.com/questions/68897076/deploy-a-spring-boot-app-with-h2-database-on-aws) 가능하긴 할 것 같은데...
- 이런 방식으로 하게 되면 일단 프로젝트가 실행될 때마다 DB가 비워지기 때문에 좋지 않을 뿐더러 테스트할 때도 반복적인 과정이 생겨서 좋지 않았을 것 같다.
> - 멘토님 덕분에 MySQL을 이용했다. 감사합니다.

<br>

#### Swagger
- 프론트와의 협업을 하면서 처음 사용해보았다. 
- 사용해보니 Controller가 개발되어야 프론트팀에 전해줄 수 있는 부분이라 어쨌든 초기 Api 명세서는 만들 필요가 있겠다고 생각했다. 
- 또한 Swagger를 사용해도 Api 명세서도 관리 및 공유에 신경써서 프론트와의 협업에 더 도움이 되어야겠다고 생각했다. 

<br>

## 느낀 점
- 진짜 너무 많이 아쉬운 프로젝트였다. 항상 혼자 작성하고 정리하던 프로젝트에서 사람들과 함께 프로젝트를 만들어 간다는 경험이 낯설고 처음이라 실수가 많았던 것 같다. 
- 코드도 깔끔하게 느껴지지 않은 부분이 많았고 마지막에 기한에 허덕이면서 작성한 코드가 많았다. (추후에 리팩토링해보자!) 
- 고생도 많이 했지만 결과가 많이 아쉽다. 미니 프로젝트에서 배우고 느낀 것을 파이널 프로젝트에 가감없이 녹여내 파이널 때는 좋은 성과도 내보고 싶다. 

<br>

## 추가 학습할 키워드
- 개발자의 협업
- AWS
- Docker
- CI / CD
- HTTPS

<br>

Reference:
- 백엔드 개발자 양성 과정 _ 패스트 캠퍼스의 미니 팀프로젝트 후 회고록입니다.