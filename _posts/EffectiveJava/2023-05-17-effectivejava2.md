---
layout: post
title: '[EFFECTIVE JAVA] ITEM 2, 생성자에 매개변수가 많다면 빌더를 고려하라'
subtitle: ''
date: 2023-05-17 11:00:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 2, 생성자에 매개변수가 많다면 빌더를 고려하라

<br>

## 대안 1, 점층적 생성자 패턴 (TeleScoping Constructor Pattern) 

- 필수 매개변수 + 선택 매개변수
- 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 호출하면 된다.
- [해당 코드 예시](https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item2/telescopingconstructor/NutritionFacts.java)

### 단점 

- 이런 생성자는 원치 않는 매개변수까지 포함하기 쉬워진다.
- 매개변수의 수가 많아지면 클라이언트 코드를 작성하거나 읽기가 더 어려워진다. 

<br>

## 대안 2, 지바빈즈 패턴 (JavaBeans Pattern)

- 매개변수가 없는 생성자로 객체를 마든 후, 세터(setter) 메소드들을 호출해 원하는 매개변수 값을 설정하는 방식
- 점층적 생성자 패턴에 비해 인스턴스를 만들기 쉽고 코드 가독성도 좋아졌다. 
- [해당 코드 예시](https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item2/javabeans/NutritionFacts.java)


### 단점

- 객체 하나를 만들기 위해서는 메소드 여러 개를 호출해야 한다.
- 객체가 완전히 생성되기 전까지는 일관성(consistency) 이 무너진 상태에 놓이게 된다.
> - 버그를 심은 코드와 그 코드 때문에 런타임에 문제를 겪는 코드가 물리적으로 떨어져 있게 되면서 디버깅도 쉽지 않아진다.
> - 결국 불변 클래스를 만들 수 없어지고, 스레드 안전성을 가지지 못하게 된다. 
- 그 방법 대안으로 freezing 방법이 있지만 (freeze 메소드) 컴파일러가 확인할 방법이 없어서 런타임 에러에 취약하게 된다.

<br>

## 대안 3, 빌더 패턴 (Builder Pattern)

- 필수 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다.
- 그 후 빌더 객체가 제공하는 일종의 세터 메소드들로 원하는 선택 매개변수들을 설정한다. 그리고 build 메소드로 객체를 얻는다. (일반적으로 불변)
- 플루언트 API(Fluent API), 메소드 연쇄(Method Chaining) : 빌더의 세터 메소드는 빌더 자신을 반환하기 때문에 연쇄적으로 호출이 가능하다.
- [해당 코드 예시](https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item2/builder/NutritionFacts.java)

### 빌더 패턴의 사용 팁

#### build 메소드가 호출하는 생성자에서 어러 매개변수의 불변식(invariant)을 검사하자
- 불변식 : 프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야 하는 조건을 말한다.
> - 가변 객체에도 불변식은 존재할 수 있고 넓게 보면 불변(어떠한 변경도 허용하지 않는다)도 불변식의 극단적인 예

#### 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다
- 각 계층의 클래스에 관련 빌더를 멤버로 정의하자, 추상 클래스(abstract class)는 추상 빌더를, 구체 클래스(concrete class)는 구체 빌더를 갖게 한다.
- 시뮬레이트한 셀프 타입(Simulated self-type) 관용구 : self 타입이 없는 자바를 위한 우회 방법으로, 추상 메소드 self를 더해 하위 클래스에서 형변환 하지 않고도 메소드 연쇄를 지원
- 공변 반환 타이핑(covariant return typing) : 하위 클래스 메소드가 상위 클래스가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능
- [해당 코드 예시](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item2/hierarchicalbuilder)

#### 빌더를 사용하면 가변인수(varangs) 매개변수를 여러 개 사용할 수 있다
- 각각을 적절한 메소드로 나눠 선언해도 되고, 메소드를 여러 번 호출하여 하나의 필드로 모을 수 있다. ex) addTopping 메소드

### 단점
- 빌더 생성 비용이 크지는 않지만 성능이 중요할 떄는 문제가 될 수 있다.
- 코드가 장황해서 매개변수 4개 이상일 때부터 사용하자
> - API는 시간이 지날수록 매개변수가 많아지니 처음부터 빌더로 시작하는 게 나을 때도 있다.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 깃헙](https://github.com/WegraLee/effective-java-3e-source-code)