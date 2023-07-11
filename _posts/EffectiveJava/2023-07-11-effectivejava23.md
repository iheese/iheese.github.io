---
layout: post
title: '[EFFECTIVE JAVA] ITEM 23, 태그 달린 클래스보다는 클래스 계층 구조를 활용하라'
subtitle: ''
date: 2023-07-11 12:30:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
---

# ITEM 23, 태그 달린 클래스보다는 클래스 계층 구조를 활용하라

<br>

## 태그 달린 클래스의 단점

- 태그 : 해당 클래스가 어떤 타입인지에 대한 정보를 담고 있는 멤버 변수를 의미한다. 

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
```

### 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다. 
- 쓸데없는 코드가 너무 많다.
> - 열거 타입, 태그 필드, switch문 등

### 태그 달린 클래스는 클래스 계층 구조를 어설프게 흉내낸 아류일 뿐이다.

<br>

## 클래스 계층 구조로 바꾸는 방법

1. 계층구조의 root가 될 추상 클래스를 정의한다.
2. 태그에 값에 따라 동작이 달라지는 메소드들을 root 클래스의 추상 메소드로 선언한다.
> - 태그 값에 상관없이 동작이 일정한 메소드는 root 클래스의 일반 메소드로 추가한다.
3. 공통으로 사용하는 데이터 필드는 모두 root 클래스로 올린다.
4. 구체 클래스에서 추상 클래스를 의미에 맞게 정의한다. 

```java
// 태그 달린 클래스를 클래스 계층 구조로 변환
// 추상 클래스 정의
abstract class Figure {
    abstract double area();
}

```


```java
// root 클래스를 확장한 구체 클래스들
class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}

```

- 계층 구조는 어떤 클래스에서 하위 클래스를 만들 때 아래와 같이 간단하게 반영하여 만들 수 있다. 

```java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

<br>

## 정리
- 태그를 사용해야하는 상황은 없다. 
- 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각해보자.
- 기존 클래스가 태그 필드를 사용하고 있다면 계층 구조로 리팩토링하는 것을 고민해보자. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
 