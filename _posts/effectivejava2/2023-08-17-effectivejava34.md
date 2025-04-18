---
layout: post
title: '[EFFECTIVE JAVA] ITEM 34, int 상수 대신 열거 타입을 사용하라'
subtitle: ''
date: 2023-08-17 11:00:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 34, int 상수 대신 열거 타입을 사용하라

<br>

## 정수 열거 패턴의 단점

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

- 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다.
> - 오렌지를 사용해야 하는데 사과를 사용해도 컴파일러는 아무런 경고를 보낼 수 없다. 
- 정수 열거 패턴을 위한 이름 공간을 지원하지 않아서 접두어를 사용해서 이름 충돌을 방지하는 방식을 사용한다.
- 평범한 상수를 나열한 것이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다. 그래서 깨지기 쉽다. 
- 같은 정수 열거 그룹에 속한 모든 상수를 한 바퀴 순회하는 방법도 마땅치 않고 그 안에 상수가 몇 개인지 알 수도 없다.

<br>

### 문자열 정수 패턴의 단점
- 정수 열거 패턴보다 더 나쁘다.

```java
public final String APPLE_FUJI = "0";
public final int APPLE_PIPPIN = "1";
public final int APPLE_GRANNY_SMITH = "2";
```

- 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하게 한다. 하드코딩한 문자열에 오타가 있어도 컴파일러는 확인할 방법이 없으니 런타임 에러가 발생한다.
- 문자열 비교에 따른 성능 저하가 일어난다. 

<br>

## 열거 타입 (ENUM)

```java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
public enum Orange {NAVEL, TEMPLE, BLOOD}
```

- 완전한 형태의 클래스(단순히 정숫값일 뿐인)라서 다른 언어의 열거 타입보다 훨씬 강력하다. 

### 열거 타입의 아이디어
- 열거 타입은 클래스이다.
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
- 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 final 이다.
- 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩 존재한다. 
> - 열거 타입은 인스턴스 통제된다.
> - 싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, 거꾸로 열거 타입은 싱글턴을 일반화한 형태라고 볼 수 있다.

### 열거 타입의 장점
- 컴타일 타임 타입 안정성을 제공한다.
> - Apple 열거 타입이 들어가야 하는 위치에 Orange를 넣을 수 없다.
- 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다. 
> - 공개되는 것은 오직 필드의 이름 뿐이라, 정수 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문이다. 
> - 열거 타입의 toString 메소드는 출력하기에 적합한 문자열을 내어준다.
- 임의의 메소드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수 있다.
- Object 메소드들, Comparable, Serializable을 구현하였고, 직렬화 형태도 웬만큼 변형을 가해도 문제없이 동작하게끔 구현되어져 있다.

<br>

## 데이터와 메소드를 갖는 열거 타입
- 상수와 관련된 데이터를 해당 상수 자체에 내재시키고 싶을 때 고차원의 추상 개념 하나를 표현할 수 있게 된다. 

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```

- 열거 타입 상수 각각을 특정 데이터에 연결짓기 위해선 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다. 
- 근본적으로 열거 타입은 불변이라 final이어야 한다.
- 필드를 private으로 두고 public 접근자 메소드를 두는 것이 낫다. 

<br>

### 열거 타입의 배열 리턴 메소드 : values 메소드

```java
public class WeightTable {
   public static void main(String[] args) {
      double earthWeight = Double.parseDouble(args[0]);
      double mass = earthWeight / Planet.EARTH.surfaceGravity();
      for (Planet p : Planet.values())
         System.out.printf("%s에서의 무게는 %f이다.%n",
                           p, p.surfaceWeight(mass));
   }
}
```

- 값들은 선언된 순서로 저장된다.

<br>

### 열거 타입 잘 사용하기
- 일반 클래스와 마찬가지로 그 기능을 클라이언트에 노출해야 할 합당한 이유가 없다면 private으로 혹은 필요하다면 package-private으로 선언하는 것이 좋다.
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 구현하는 것이 좋다.
- 특정 톱레벨 클래스에서만 쓰이면 해당 클래스이 멤버 클래스로 만드는 것이 좋다.

<br>

## 상수별 메소드 구현
- 상수마다 동작이 달라져야 하는 상황에 만약 switch 문을 이용해 분기한다면?

```java
public static Operation inverse(Operation op) {
        switch(op) {
            case PLUS:   return Operation.MINUS;
            case MINUS:  return Operation.PLUS;
            case TIMES:  return Operation.DIVIDE;
            case DIVIDE: return Operation.TIMES;

            default:  throw new AssertionError("Unknown op: " + op);
        }
    }
```

- 위는 깨지기 쉬운 코드이다. 새로운 상수를 추가하면 해당 case도 추가해야 한다.
- 아래 코드는 상수별로 다르게 동작하는 코드를 구현하는 더 나은 수단을 제공한다.

### 상수별 메소드 구현 (constant-specific method implementation)

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

- 열거 타입에 추상 메소드를 선언하고 각 상수별 클래스 몸체(constant-specific class body) 를 각각 재정의하는 방법이다. 

<br>

### valueOf(String) 메소드
- 상수의 이름을 받아 해당 상수를 반환해주는 메소드

<br>

### fromString 메소드 제공
- toString 메소드를 재정의 할 때 fromString 메소드도 고려하여 함께 제공하는 것도 좋다.
- fromString 메소드는 toString이 반환하는 문자열을 해당 열거 타입 상수로 반환해주는 메소드이다. 

```java
    @Override public String toString() { return symbol; }

    private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                    toMap(Object::toString, e -> e));
 
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
```

- Operation 상수가 stringToEnum 맵에 추가되는 시점은 열거 타입 상수 생ㅇ성 후 정적 필드가 초기화되었을 때다.
- 열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없다. >  컴파일 오류
- 열거 타입의 정적 필드 중 열거 타입 생서자에서 접근할 수 있는 것은 상수 변수 뿐이다.
> - 열거 타입 생성자가 실행되는 시점에는 정적 필드들이 초기화 전이라 자기 자신을 추가하지 못하게 하는 제약이 꼭 필요하다.
- 열거 타입 생성자에서 같은 열거 타입의 다른 상수에도 접근할 수 없다. 
> - 열거 타입의 각 상수는 해당 열거 타입의 인스턴스를 public static final 필드로 선언, 다른 형제 상수도 static 이므로 열거 타입 생성자에서 정적 필드에 접근할 수 없다는 제약이 적용

<br>

## 전략 열거 타입 패턴
- 열거 타입 상수까리 메소드를 공유한다면 사용해볼 방식이다.

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    //전략 열거 타입
   enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
   public static void main(String[] args) {
        for (PayrollDay day : values())
            System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
    }
}
```

- 추가하려는 메소드가 의미상 열거 타입에 속하는 경우 사용해볼 수 있다.
- PayrollDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여 안전하고 유연하게 처리한다. 
- 추가하려는 메소드가 의미상 열거 타입에 속하지 않으면 switch 를 적용한다. 

<br>

## 열거 타입을 언제 사용해야 할까?
- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자
> - EX) 태양계 행성, 한 주의 요일, 체스 말
- 열거 타입에 정의된 상수 갯수가 영원히 고정 불변일 필요는 없다.
> - EX) 메뉴 아이템, 연산 코드, 명령줄 플래그
> - 나중에 상수가 추가되도 바이너리 수준에서 호환되도록 설계되었다.
- 추가적으로 열거 타입의 성능는 정수 상수와 변반 다르지 않다. 

<br>

## 정리
- 열거 타입은 확실히 정수 상수보다 낫다. 더 읽기 쉽고 안전하고 강력하다.
- 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작할 때는 명시적 생성자, 메소드가 필요할 수 있다.
- 하나의 메소드가 상수별로 다르게 동작해야 할 때는 switch 문 대신 상수별 메소드 구현을 사용하자.
- 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
