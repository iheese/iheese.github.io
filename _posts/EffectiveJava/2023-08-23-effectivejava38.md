---
layout: post
title: '[EFFECTIVE JAVA] ITEM 38, 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라'
subtitle: ''
date: 2023-08-23 11:50:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 38, 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

<br>

## 열거 타입 확장은 좋은 생각이 아니다
- 열거 타입은 타입 안전 열거 패턴(typesafe enum pattern)보다 우수하다.
- 하지만 타입 안전 열거 패턴은 확장 가능하나, 열거 패턴은 그럴 수 없다. 

```java
// 타입 안전 열거 패턴의 예시
public class TypeSafeEnum {
    private final String type;
    private TypeSafeEnum(String type) {
        this.type = type;
    } 

    public String toString() {
        return type;
    }

    public static final TypeSafeEnum typeSafeEnum1 = new TypeSafeEnum("type1");
    public static final TypeSafeEnum typeSafeEnum2 = new TypeSafeEnum("type2");
}
```

- 확장 타입의 원소는 기반 타입의 원소 취급하지만 반대는 아니다.
- 기반 타입과 확장된 타입의 원소를 순회할 방법도 마땅치 않다.
- 확장성을 높이려면 고려할 요소가 많아져 설계와 구현이 어려워진다. 

<br>

#### 확장할 수 있는 열거 타입이 어울리는 쓰임
- 연산 코드 (operation code, opcode) : 연산 코드의 각 원소는 특정 기계가 수행하는 연산을 뜻한다. 

<br>

## 열거 타입의 확장
- 열거 타입이 임의의 인터페이스를 구현할 수 있다는 사실을 이용하자.

```java
// 연산 코드용 인터페이스 정의
public interface Operation {
    double apply(double x, double y);
}
```

```java
public enum BasicOperation implements Operation {
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

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

- 위처럼 하면 Operation은 확장 가능하고, 이 인터페이스를 연산의 타입으로 사용하면 된다. 
- Operation을 구현한 또 다른 열거 타입으로 대체할 수 있게 된다. 

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

<br>

## 열거 타입 확장 시 사용법

#### 한정적 타입 토큰 

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}
private static <T extends Enum<T> & Operation> void test(
        Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```

- ExtendedOperation의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다. 
> - 해당 class 리터럴은 한정적 타입 토큰 역할을 한다. 
- `<T extends Enum<T> & Operation>` : Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 뜻

<br>

#### 한정적 와일드 카드 타입

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}
private static void test(Collection<? extends Operation> opSet,
                            double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```

- 그나마 덜 복잡해지고 유연해졌다. 여러 구현 타입의 연산을 조합해 호출할 수 있게 되었다. 
- 하지만 특정 연산에서 EnumSet, EnumMap을 사용하지 못한다. 

<br>

## 인터페이스를 이용해 확장 가능한 열거 타입을 흉내내는 법의 문제점
- 열거 타입끼리 상속할 수 없다.
- 아무 상태에 의존하지 않는 경우 디폴트 구현을 이용해 인터페이스에 추가할 수 있다. 
- 하지만 Operation에 연산 기호를 저장하고 찾는 로직이 BasicOperation, ExtendedOperation 모두에 들어가야 한다. 
> - EX) java.nio.file.LinkOption 열거 타입은 CopyOption, OpenOption 인터페이스를 구현했다. 

<br>

## 정리
- 열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 사용해 같은 효과를 낼 수 있다. 
- API가 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
