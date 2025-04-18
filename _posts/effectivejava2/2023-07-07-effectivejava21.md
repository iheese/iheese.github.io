---
layout: post
title: '[EFFECTIVE JAVA] ITEM 21, 인터페이스는 구현하는 쪽을 생각해 설계하라'
subtitle: ''
date: 2023-07-07 12:30:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 21, 인터페이스는 구현하는 쪽을 생각해 설계하라

<br>

## 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메소드를 작성하기란 어려운 법이다
- 디폴트 메소드는 구현 클래스에 대해 아무것도 모른 채 합의 없이 무작정 삽입될 뿐이다.
- Java 8에서는 핵심 컬렉션 인터페이스들에 다수의 디폴트 메소드가 추가되었다. 
> - 람다에서 활용하기 위해서
- 범용적으로 구현되어있긴 하지만 현존하는 모든 컬렉션 구현체와 어울러지는 것은 아니다. 

<br>

### SynchronizedCollection
- org.apche.commons.collection4.collection.SynchronizedCollection
- java.util의 Collection.synchronnizedCollection 정적 팩터리 메소드가 반환하는 클래스와 비슷하다.
- 아파치 버전에서는 클라이언트가 제공한 객체로 락을 거는 능력을 추가로 제공한다.
- 즉, 모든 메소드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스이다. 
- removeIf 메소드는 재정의 되어있지 않았다. 
> - java8과 함께 사용한다면(디폴트 구현을 물려받는다면) 모든 메소드 호출을 알아서 동기화해주지 않는다.

<br>

## 디폴트 메소드는 (컴파일에 성공하더라도) 기존 구현체에 런타임 오류를 일으킬 수 있다
- 기존 인터페이스에 디폴트 메소드로 새 메소드를 추가하는 일은 꼭 필요하지 않으면 지양하자.
- 하지만 새로운 인터페이스를 만드는 경우는 디폴트 메소드가 아주 유용한 수단이다. 
- 디폴트 메소드는 기존의 메소드의 시그니처를 수정하는 용도가 아님을 명심해야 한다.
> - 기존 클라이언트에 큰 영향이 갈 수 있다.

<br>

## 인터페이스 설계시 주의가 필요하다.

- 새로운 인터페이스 설계시 검증을 위해 최소 3가지는 구현해봐야 한다.
- 인터페이스의 인스턴스를 다양한 작업에 활용하는 클라이언트도 여러 개 만들어봐야 한다..
- 인터페이스를 릴리즈한 후라도 결함을 수정하는 게 가능한 경우도 있겠지만, 절대 그 가능성에 기대선 안된다.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
