---
layout: post
title: '[EFFECTIVE JAVA] ITEM 36, 비트 필드 대신 EnumSet을 사용하라'
subtitle: ''
date: 2023-08-21 11:40:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 36, 비트 필드 대신 EnumSet을 사용하라

<br>

## 비트 필드 열거 상수
- 열거한 값이 주로 집합으로 사용될 경우 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용하였다.

```java
public class Text{
  public static final int STYLE_BOLD = 1 << 0; // 1
  public static final int STYLE_ITALIC = 1 << 1; // 2
  public static final int STYLE_UNDERLINE = 1 << 2; // 4
  public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR 한 값
  public void applyStyles(int styles){...}
}
```

- 비트 필드 : 비트별 OR를 사용해 여러 상수를 하나로 모은 집합

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

- 비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다.

<br>

## 비트 필드의 문제점
- 정수 열거 상수의 단점을 그대로 가지고 있다.
- 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기 어렵다
- 비트 필드 하나의 녹아 있는 모든 원소를 순회하기도 까다롭다.
- 최대 몇 비트가 빌요한지 API 작성 시 미리 예측해서 적절한 타입을 선택해야 한다.
> - API를 수정하기 않고는 비트 수를 더 늘릴 수 없기 때문이다.(int: 32비트, long: 64비트)

<br>

## 비트 필드의 대안 : EnumSet
- 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.
> - Set 인터페이스를 완벽히 구현한다.
> - 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다.
- 내부도 비트 벡터로 구현되어 있어 비트 필드에 비견한 성능을 보여준다.
> - 원소가 64개 이하라면 즉 EnumSet 전체를 long 변수 하나로 표현할 수 있다.
- removeAll, retainAll 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.

```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }

    // 사용 예
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

- EnumSet을 건네리라 짐작되어도 이왕이면 인터페이스로 받아서 특이한 클라이언트가 다른 Set을 넣는 것을 대비하여 처리하자.

<br>

## 정리
- 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다 해도 비트 필드를 사용할 이유는 없다.
- EnumSet 클래스가 비트 필드 수준의 명료함과 성능을 제공하고 열거 타입의 장점까지 선사한다.
- 불변 EnumSet을 만들 수 없는 것은 단점이다.
> - 자바 9까지는 그렇다.
> - Collections.unmodifiableSet으로 Enum을 감싸 사용할 수 있다.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
