---
layout: post
title: '[EFFECTIVE JAVA] ITEM 42, 익명 클래스보다는 람다를 사용하라'
subtitle: ''
date: 2023-10-16 12:20:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# 7장 람다와 스트림

# ITEM 42, 익명 클래스보다는 람다를 사용하라

<br>

## 익명 클래스
- 함수 객체 : 자바에서 함수 타입을 표현할 때 추상 메소드를 하나만 담은 인터페이스(드물게는 추상 클래스)를 사용하고, 해당 인터페이스의 인스턴스를 함수 객체라고 한다.
- JDK 1.1 등장 후 함수 객체를 만드는 주요 수단은 익명 클래스이다.

```java
Collections.sort(words, new Comparator<String>() {
            public int compare(String s1, String s2) {
                return Integer.compare(s1.length(), s2.length());
            }
        });
```

- Comparator 인터페이스가 정렬을 담당하는 추상 전략, 내부 구체적인 전략은 익명 클래스로 구현했다. 
- 이는 코드가 너무 길어 함수형 프로그래밍에 적합하지 않다. 

<br>

## 람다식
- 자바 8 에 와서 함수 객체를 람다식을 사용해 짧게 만들 수 있게 되었다. 

```java
Collections.sort(words,
        (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

- 람다의 타입 : Comparator<String>, 매개변수(s1, s2)의 타입 : String, 반환값의 타입 : int
- 컴파일러가 문맥을 살펴 타입을 추론해준다. 
- 상황에 따라 컴파일러가 타입울 결정하지 못할 때만 명시해주면 된다. 람다의 모든 매개변수 타입은 생략하는 것이 편하다.
> - 컴파일러가 타입을 추론하는 데 필요하는 타입 정보 대부분은 제네릭에서 얻기 때문에 제네릭을 사용하는 것은 중요하다. 

<br>

```java
Collections.sort(words, comparingInt(String::length));
```

- 람다 자리에 비교자 생성 메서드를 사용해 더 간결하게 만들었다.

<br>

```java
words.sort(comparingInt(String::length));
```

- 자바 8 에 List 인터페이스에 추가된 sort 메소드를 이용하면 더 짧게 만들 수 있다.

<br>

## 열거 타입에서 람다 사용

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    public abstract double apply(double x, double y); //추상메소드
}
```

- 상수별 클래스 몸체와 데이터를 사용한 열거 타입이다. 

```java
public enum Operation {
    PLUS  ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override public String toString() { return symbol; }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

- 람다를 사용해 열거 타입의 인스턴스 필드를 이용하는 방식이다. 
> - DoubleBinaryOperator : java.util.function 이 제공하는 다양한 함수 인터페이스로, double 타입 인수 2개를 받아 double 타입 결과를 돌려준다.

#### 람다는 이름이 없고, 문서화도 못한다. 따라서 코드 자체로 동작이 명확하게 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.
- 람다는 세 줄이 넘어가면 가독성이 매우 나빠진다. 너무 길거나 읽기 어렵다면 줄이거나 람다를 사용하지 않는 쪽으로 리팩토링하는 것이 좋다.
- 열거 타입 생성자에 넘겨지는 인수들은 컴파일 타임에 추론된다. 따라서 열거 타입 생성자 안의 람다는 열거 타입 인스턴스 멤버에 접근할 수 없다.
> - 인스턴스는 런타임에 만들어지기 때문이다. 
- 상수별 동작을 단 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메소드를 사용해야만 하는 상황이람ㄴ 상수별 클래스 몸체를 사용해야 한다.

<br>

## 람다로 대체할 수 없는 것
- 람다는 함수형 인터페이스에서만 사용된다.

### 예시
- 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니 익명 클래스를 써야 한다. 
- 추상 메소드가 여러 개인 인터페이스의 인스턴스를 만들 때도 익명 클래스를 쓸 수 있다.
- 람다는 자신을 참조할 수 없다. 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스로 써야 한다.
> - 람다의 this는 바깥 인스턴스를 가리키고, 익명 클래스의 this는 익명 클래스 인스턴스 자신을 가리킨다.

### 람다, 익명 클래스의 인스턴스를 직렬화하는 일은 극히 삼가야 한다.
- 람다, 익명 클래스 모두 직렬화 형태가 가상머신 구현별로 다를 수 있다. 
- 직렬화해야만 하는 함수 객체가 있다면 private 정적 중첩 클래스의 인스턴스를 사용하는 것이 좋다. 
> - Serializable 인터페이스 구현하여 클래스 작성하기

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
