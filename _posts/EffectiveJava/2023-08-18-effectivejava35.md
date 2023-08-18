---
layout: post
title: '[EFFECTIVE JAVA] ITEM 35, ordinal 메소드 대신 인스턴스 필드를 사용하라'
subtitle: ''
date: 2023-08-18 10:30:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 35, ordinal 메소드 대신 인스턴스 필드를 사용하라

<br>

## ordinal 메소드
- 대부분 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응된다. 
- ordinal 메소드 : 해당 상수가 열거 타입 내에서 몇 번째 위치에 있는지 반환한다.

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;
 
    public int numberOfMusicians() { return ordianl() + 1; }
}
```

- 위는 유지보수하기가 매우 어려운 코드이다.
- 상수 선언 순서를 바꾸는 순간 numberOfMusicians 가 오작동한다. 
- 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다. 
> - 정수값이 8 인 것은 하나만 넣을 수 있다.
- 값을 중간에 비워둘 수 없다. 
> - 정수값 12인 것을 만들고 싶다면 중간의 11을 넣어야 하므로 더미 상수를 만들어야 하는 단점이 있다. 
> - 이는 코드가 더러워지고 쓰이지 않는 값이 많아져 실용성이 매우 떨어진다. 

<br>

## 해결책

### 열거 타입 상수에 연결된 값은 ordinal 메소드로 얻지 말고 인스턴스 필드에 저장하자

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

- Enum API 문서 왈 : 대부분 프로그래머는 이 메서드를 사용할 일이 없고, EnumSet, EnumMap 같이 열거 타입 기반의 범용 자료구조에 사용될 목적으로 설계되었다.
- 따라서 위와 같은 목적이 아니라면 절대 사용하지 말자!

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
