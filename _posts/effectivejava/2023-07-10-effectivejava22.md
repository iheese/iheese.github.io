---
layout: post
title: '[EFFECTIVE JAVA] ITEM 22, 인터페이스는 타입을 정의하는 용도로만 사용하라'
subtitle: ''
date: 2023-07-10 12:00:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
---

# ITEM 22, 인터페이스는 타입을 정의하는 용도로만 사용하라

<br>

## 인터페이스의 역할
- 자신이 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다. 
> - 즉, 클래스가 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 말해주는 것과 같다.

<br>

## 상수 인터페이스
- 위 지침에 맞지 않는 예시이다.
- 메소드 없이 상수를 뜻하는 static final 필드만 가득 찬 인터페이스를 의미한다.
- 정규화된 이름을 쓰는 것을 피하고자 이를 사용한다.

### 상수 인터페이스의 안티 패턴은 인터페이스를 잘못 사용한 예이다.

```java
// 상수 인터페이스 안티패턴 - 사용금지! 
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니 내부 구현에 해당한다.
- 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스 API로 노출하는 행위이다. 
> - 클라이언트 코드가 내부 구현에 종속되게 하고 혼란을 준다.
- 인터페이스의 필드는 public 접근제어자와 관계없이 static final로 역할한다. 
- java.io.ObjectStreamConstants 등의 자바의 상수 인터페이스 사용은 지양하자

<br>

## 상수를 올바르게 공개하는 방법

### 특정 클래스나 인터페이스와 강하게 연관된 상수
- 모든 숫자 기본 타입의 박싱 클래스
- EX) `Integer.MIN_VALUE`, `Integer.MAX_VALUE`, `Double.MIN_VALUE`, `Double.MAX_VALUE`

### Enum
- 열거 타입과 적합하면 열거 타입으로 생성한다

### 인스턴스화할 수 없는 유틸리티 클래스

```java
public class PhysicalConstants {
  private PhysicalConstants() { }  // 인스턴스화 방지

  // 아보가드로 수 (1/몰)
  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

  // 볼츠만 상수 (J/K)
  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;

  // 전자 질량 (kg)
  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```

<br>

## 정리
- 인터페이스는 타입을 정의하는 용도로만 사용하고, 상수 공개용으로는 사용하지 말아야 한다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
