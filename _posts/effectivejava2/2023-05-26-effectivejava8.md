---
layout: post
title: '[EFFECTIVE JAVA] ITEM 8, finalizer와 cleaner 사용을 피하라'
subtitle: ''
date: 2023-05-26 13:00:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 8, finalizer와 cleaner 사용을 피하라

<br>

## 자바의 객체 소멸자

- finalizer : 예측할 수없고, 상황에 따라 위험할 수있어 일반적으로 불필요하다. "쓰지말자!"
> - 오작동, 낮은 성능, 이식성 문제의 원인 
- cleaner : finalizer의 대안, finalizer보다는 덜 위험하지만 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다. 

- C++의 파괴자(destructor) = 자바의 try-with-resources, try-finally

<br>

##  finalizer와 cleaner 사용하지 말아야 하는 이유

### 즉시 수행된다는 보장이 없다.
- 객체에 접근 할 수 없게 된 후  finalizer와 cleaner가 실행되기까지 얼마나 걸리는지 알 수 없다. 
- 즉 finalizer와 cleaner가 제때 실행되어야 하는 작업은 절대 할 수 없다. 
- finalizer와 cleaner가 신속하게 수행될지는 가비지 컬렉터의 알고리즘에 따라 다르며 구현마다 천차만별이다.

### 수행 여부조차 보장하지 않는다.
- 접근할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수 있다.
- 상태를 영구적으로 수정하는 작업에서는 절대 finalizer와 cleaner를 의존하면 안된다. 
> - EX) 데이터베이스 같은 공유 자원의 영구 락 해제 > 분산 시스템 전체가 서서히 멈춘다.
- System.gc, System.runFinalization 메서드에 현혹되지 말자 : 실행될 가능성은 높여주나 보장해주진않는다.
- System.runFinalizerOnExit, Runtime.runFinalizerOnExit : 심각한 결함

###  finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았어도 그 순간 종료된다. 
- 잡지 못한 예외 때문에 해당 객체는 자칫 마무리가 덜 된 상태로 남을 수 있다.
- 다른 스레드가 훼손된 객체를 사용하려 한다면 어떻게 동작할지 예측할 수 없다.

### finalizer와 cleaner는 심각한 성능 문제도 동반한다.
- try-with-resource로 AutoCloseable 객체를 생성하고 GC가 수거할 때까지 12ns가 걸렸다.
- finalizer룰 사용해서 객체를 수거하면 550ns가 걸린다. 
- GC의 효율이 매우 떨어진다. 

### finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다. 
- finalizer 공격원리 : 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행되게 한다. 
> - finalizer는 정적 필드에 자신의 참조를 할당하여 GC가 수집하지 못하게 막을 수 있다. 
> - 일그러진 객체가 만들어지면, 해당 객체의 메소드를 호출하여 허용되지 않았을 작업을 수행할 수 있게 된다. 
- 객체 생성을 막으려면 생성자에서 예외를 던지면 되지만 finalizer는 소용이 없다.
> - 방어 방법 : 아무 일도 하지 않는 finalizer 메소드를 만들고 final 로 선언한다. 

<br>

## finalizer와 cleaner의 대안
- AutoCloseable을 구현하고, 클라이언트에서 인스턴스를 다 쓰면 close 메소드를 호출해 준다. 
- 예외가 발생해도 제대로 종료되도록 try-with-resources를 사용해야 한다. 

<br>

## finalizer와 cleaner의 쓰임새

### 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할
- 안전망 역할의 finalizer를 작성할 때는 그럴만한 값어치가 있는지 심사숙고해봐야한다.
> - EX) 자바의 FileInputStream, FileOutputStream, ThreadPoolExecutor

### 네이티브 피어와 연결된 객체
- 네이티브 피어(native peer) : 일반 자바 객체가 네이티브 메서드를 통해 기능을 위한 네이티브 객체를 말한다. 
> - 자바 객체가 아니므로 GC가 존재를 알지 못한다.
- 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때만 사용
> - 성능 저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 close() 메서드를 사용해야 한다.

<br>

## 핵심 정리
- cleaner(자바8까지 finalizer) 는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자
- 위의 경우라도 불확실성과 성능 저하에 주의하자

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 깃헙](https://github.com/WegraLee/effective-java-3e-source-code)