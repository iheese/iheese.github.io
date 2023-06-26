---
layout: post
title: '[EFFECTIVE JAVA] ITEM 16, public 클래스에서는 public 필드가 아닌 접근자 메소드를 사용하라'
subtitle: ''
date: 2023-06-26 12:00:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 16, public 클래스에서는 public 필드가 아닌 접근자 메소드를 사용하라

<br>

## 가변 필드를 public으로 접근할 수 있게 하면 안된다.
- 캡슐화의 이점을 제공하지 못한다.
- API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
- 불변식을 보장할 수 없다.
- 외부에서 필드에 접근할 때 부수 작업을 수행할 필요도 없다.

<br>

### 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공하자
- 필드를 모두 private 으로 바꾸고 public getter 접근자를 제공하자
- 이를 통해 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

<br>

### package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다해도 하등의 문제가 없다.
- 이 방법이 클라이언트 코드 면에서나 접근자 방식보다 깔끔하다.
- package-private : 패키지 바깥 코드는 전혀 손대지 않고 데아터 표현 방식을 바꿀 수 있다.
- private 중첩 클래스 : 수정 범위다 더 좁아짐, 이 클래스를 포함하는 외부 클래스까지 제한된다. 

<br>

## public 클래스의 불변 필드
- 규칙을 어기는 자바 플랫폼 라이브러리 :  java.awt.package 의 Point, Dimension 클래스
- 직접 노출할 때의 단점이 조금 줄어들지만 좋은 생각은 아니다.

- 단점 : API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업이 필요한 단점이 존재한다.
- 장점 : 불변식을 보장한다. 

<br>

## 결론
- public 클래스의 절대 가변 필드는 직접 노출해서는 안된다.
- 불변 필드는 노출하는 것이 덜 위험하지만 안심할 수 없다.
- package-private(default) 클래스나 private 중접 클래스에서는 필드를 노출하는 것이 나을 때도 있다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
