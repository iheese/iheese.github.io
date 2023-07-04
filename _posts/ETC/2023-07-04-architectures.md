---
layout: post
title: '[ETC] Monolithic vs SOA vs MSA'
subtitle: '모놀리틱(Monolithic), SOA(Service Oriented Architecture), MSA(Micro Service Architecture)'
date: 2023-07-04 12:00:00 +0900
categories: 'ETC'
background: 'https://github.com/iheese/TIL/assets/88040158/1caf8cba-14fd-4c5c-8cea-4afc8ba8db45'
published: true
---

# Monolithic vs SOA vs MSA

![스크린샷 2023-07-04 오전 10 13 34](https://github.com/iheese/TIL/assets/88040158/a51bb78d-c4a5-49e8-a5f5-d5044956f6c5)

- Monolthic : 하나의 프로젝트
- SOA(Service Oriented Architecture) : 여러 서비스, 하나의 버스
- MSA(Micro Service Architecture) : 독립된 여러 서비스 

<br>

# Monolithic Architecture

![1_Tiizur0VlvZlJSIgADsp4w](https://github.com/iheese/TIL/assets/88040158/a1c6496c-8d06-4a02-86c7-ea9881fb1260)

- Monolithic : 단단히 짜 하나로 되어 있는
- 모놀리틱 아키텍처는 하나의 프로젝트에 코드가 모여 있고 하나의 파일로 구성된다. 

## 장점
- 로컬 환경에서 개발하고 테스트하기 매우 편리하다.
> - 모든 코드와 테스트 코드까지 하나의 어플리케이션 안에 존재하기 때문이다.
- 하나의 묶음으로 구성되어 있기 때문에 배포도 간편하다.
- 인프라의 구성과 운용이 간편하다. 

## 단점
- 서비스가 커짐에 따라 빌드, 배포, 서버 가동 시간이 점점 늘어난다. 
> - 잦은 배포시 긴 배포 시간은 문제가 될 수 있다.
- 서비스 간 의존성이 존재하기 때문에 시스템 일부만 수정해도 전체 어플리케이션의 빌드, 테스트, 배포 과정을 다 거쳐야 한다. 
- 부분적 스케일 아웃이 어렵다.
> - 성능을 늘리기 위해선 스케일 업만 가능하다. (스케일 아웃: 서버의 갯수를 늘려 전체 성능 올리기, 스케일 인: 서버의 내부 성능을 높여 전체 성능 높이기)
- 안정성이 떨어진다.
> - 부분 장애가 전체 서비스 장애로 퍼질 가능성이 매우 크다.
- 서비스가 커짐에 따라 히스토리, 이슈 파악이 매우 어려워진다.
- 코드 작은 일부분을 수정했을 때 미칠 영향력을 파악하기 쉽지 않다.
> - EX) A 서비스에 사용되는 유틸 기능 코드 하나를 변경했는데, 사실 그 유틸 기능이 B, C, D 서비스에 널리 퍼져 사용되고 다른 코드 깊숙이 있었다면 파악하기 힘들다. 
- 하나의 프레임 워크, 언어에 종속적이다. 

<br>

# SOA(Service Oriented Architecture)

![esb_architecture](https://github.com/iheese/TIL/assets/88040158/ddb4c52d-b43f-4fe3-beb6-baf3792a15f3)

- SOA(Service Oriented Architecture) : 서비스 지향 아키텍처
- SOA는 기존의 어플리케이션의 기능들을 비지니스적인 의미를 가지는 기능 단위로 묶어서 표준화된 호출 인터페이스를 통해 서비스로 구현하고, 이 서비스들을 기업의 업무에 따라 어플리케이션을 구성하는 소프트웨어 개발 아키텍처를 의미한다.
- 공통된 서비스를 ESB(Enterprise Service Bus)에 모아 사업 측면에서 공통의 기능 집합을 통해 서비스를 제공한다.
> - ESB(Enterprise Service Bus) : 서비스를 컴포넌트화된 논리적 집합으로 묶는 미들웨어로 이벤트 및 서비스에 대한 요청과 처리를 중개하여 전체 스트럭처에 분포하게 된다. 
- 모듈의 의존성은 줄이되 모듈 내에서 공유할 수 있는 것은 공유하자는 정책이므로 ESB를 도입하였다. 

## 장점
- ESB 를 사용하여 서비스 단위로 분리되다 보니 결합도가 낮아지는(느슨한) 아키텍처가 된다.
- 각 서비스 모듈이 독립적으로 운영되고 재사용성이 높아진다.
- ESB 스트럭처를 잘 이용한다면 확장성과 유연성이 증가한다는 특징이 있다.

## 단점
- ESB 도 하나의 DB를 사용한다는 의존성이 문제일 수 있다. 

<br>

# MSA(Micro Service Architecture)

![imagesoa](https://github.com/iheese/TIL/assets/88040158/b907586c-3959-4ab0-b172-41e63c4228cf)

![imagemsa](https://github.com/iheese/TIL/assets/88040158/db4ebe5a-18ed-4af0-a964-704ed6934a4a)

SOA vs MSA

- MSA(Micro Service Architecture) : 아주 작은 서비스 아키텍처
- SOA 보다 더 세분화하여 함께 작동하며, 작은 모놀리틱 아키텍처 여러 개가 최소한의 의존성을 가지며 큰 서비스를 구축하는 형태라고 볼 수도 있다. 
- 독립적인 역할을 수행하는 작은 단위의 서비스로 분리하고, 서로 서비스가 API를 통해 통신할 수 있도록 설계하는 아키텍처이다. 

## 장점
- 서비스가 개별적으로 나누어져 있기 때문에 하나의 서비스에 문제가 생겨도 다른 서비스에 영향을 줄 가능성이 낮다.
- 필요에 따라 개별 서비스에 새로운 기술 스택을 추가할 수 있는 유연성을 가진다.
- 각 서비스마다 프레임 워크, 언어가 달라도 된다. 즉 종속적이지 않다.
- 트래픽이 많은 서비스만 개별적으로 확장 가능하여 경제적일 수 있다.
> - 스케일 업, 스케일 아웃이 가능하다. 
- 작은 크기의 서비스들이어서 빌드, 배포가 빠르다.
- 프로그래머들이 코드와 히스토리를 파악하기 용이하다.
> - 해당 어플리케이션의 역할이 명확하기 때문에 코드 파악이 용이한 편이다. 

## 단점
- 각 서비스 간 API 통신, 실패 처리 등에 대한 고려가 필요하다.
- 여러 DB를 통해 데이터를 처리하다보니 트랜잭션 처리가 중요해진다.
- 서비스를 테스트할 때 비교적 어렵다.
> - 여러 어플리케이션들이 얽혀 있다면 모두 실행하여 테스트하거나 다른 방법을 도입하여 테스트하여야 한다.
> - 특히 통합 테스트는 운영환경과 똑같은 환경으로 가져가 테스트하는데 어려움이 있다.
- 각 어플리케이션의 배포 순서가 중요할 수 있다.
- MSA 를 구축하고 파악하는 일은 쉬운 일이 아니다. 
- 많은 어플리케이션을 구축하고 관리하는데 비용, 인력 비용이 큰 편이다. 

<br>

# 정리
- 결합도 : Monolithic > SOA > MSA
- 확장성 : MSA > SOA > Monolithic
- 바용 : MSA > SOA > Monolithic
- 기술적 난이도 : MSA >= SOA > Monolithic

<br>

#### 어떤 아키텍처가 최고라고는 할 수 없을 것 같고 조직의 상황에 맞는 아키텍처를 사용하는 것이 좋은 것 같다. 

<br>

Reference:

- [[마이크로서비스] Monolithic vs SOA vs Microservice, 다양한 아키텍처와의 비교 _ Wonit](https://wonit.tistory.com/487)
- [Monolithic img](https://medium.com/design-microservices-architecture-with-patterns/monolithic-architecture-is-still-worth-at-2021-98bfc112dc24)
- [SOA img](https://www.hcltech.com/blogs/everything-you-need-know-about-enterprise-service-bus-esb)
- [SOA, MSA img _ 리니 LGCNSer](https://blog.naver.com/PostView.nhn?blogId=stmshra&logNo=221446919085&categoryNo=80&parentCategoryNo=0&viewDate=&currentPage=3&postListTopCurrentPage=&from=postList&userTopListOpen=true&userTopListCount=5&userTopListManageOpen=false&userTopListCurrentPage=3)
- [배경 img _ 오늘의집 MSA](https://www.bucketplace.com/post/2021-11-19-%EC%98%A4%EB%8A%98%EC%9D%98%EC%A7%91-msa-%EC%97%AC%EC%A0%95-part-1-%EC%8B%9C%EC%9E%91/)