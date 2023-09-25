---
layout: post
title: '[JAVA, SPRING WEBFLUX] SPRING WEBFLUX 란? '
subtitle: ''
date: 2023-09-25 12:00:00 +0900
categories: 'SPRING'
background: '/img/posts/etc/spring.jpg'
---

# Spring WebFlux
- Spring 5 부터 새롭게 추가된 모듈이며, 클라이언트/서버 구조에서 리액티브(reactive) 어플리케이션 개발을 위한 논블로킹 리액티브 스트림을 지원한다. 
> - 리액티브(reactive) 프로그래밍 : 데이터 스트림과 변경 사항 전파를 중심으로 하는 비동기 프로그래밍, 즉 데이터 흐름에 초점이 맞춰져 있고, 데이터를 비동기적으로 처리하며 이벤트 기반의 아키텍처로 실시간으로 데이터의 변화에 반응할 수 있는 프로그래밍이다. 
> - 논블로킹(non blocking) : A함수가 B함수를 호출해도 제어권은 그대로 자신이 가지고 있는다. 그래서 B 함수가 호출되어도 A함수는 계속 실행된다. 
- 적은 양의 쓰레드로 동시성을 처리하고, 보다 적은 하드웨어 자원으로 동시성을 처리하기 위한 위한 논블로킹(non-blocking) web 스택이 필요했기 때문에 만들어졌다.
- 함수형 프로그래밍 때문에 생기게 되었다. 자바 8 에서 함수형 API를 위한 람다식이 추가되었는데 이는 논블로킹 어플리케이션을 만들 때도 요긴하게 쓰인다. 비동기 로직의 선언적인 합성을 가능하게 하면서 연속적인 전달 스타일의 API를 작성할 수 있게 되었다.  

<br>

## Spring WebFlux VS Spring MVC

#### 공통점 
- @Controller
- Reactive 클라이언트
- Tomcat, Jetty, Undertow 같은 서버에서 실행할 수 있다.

#### 차이점 
- Spring MVC :
> - 명령형 논리(명령을 순서대로 처리), JDBC, JPA
> - 하나의 요청에 대해 하나의 쓰레드가 사용된다. 다량의 요청에 대비 쓰레드 풀을 생성해놓고 각 요청마다 쓰레드를 할당하여 처리한다.
> - 1 request : 1 thread, sync + blocking
- Spring WebFlux :
> - 기능적 엔드 포인트, 이벤트 루프, 동시성 모델
> - 논블로킹과 고정된 쓰레드 수로 모든 요청을 처리한다. 서버는 쓰레드 한 개로 운영하며, 디폴트로 CPU 코어 수 갯수의 쓰레드를 가진 워커 풀을 생성하여 워커 풀 내 쓰레드로 요청을 처리한다. 
> - many request : 1 thread, async + non-blocking


<br>

Reference:

- [WebFlux란 무엇인가? _ 코딩무니](https://devmoony.tistory.com/174)
