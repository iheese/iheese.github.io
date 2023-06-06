---
layout: post
title: '[EFFECTIVE JAVA] ITEM 10, equals는 일반 규약을 지켜 재정의하라'
subtitle: ''
date: 2023-06-06 16:00:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 10, equals는 일반 규약을 지켜 재정의하라

<br>

## equals 메소드를 재정의하지 않아야 할 상황

#### 각 인스턴스가 본질적으로 고유할 때
- 값을 표현하는 것이 아니라 동작하는 개체를 표현하는 클래스
- EX) Thread, Bean에 등록되는 Controller, Service, Repository

<br>

#### 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없을 때
- 논리적 동치성 검사의 예시: java.util.regex.Pattern의 두 인스턴스가 같은 정규표현식을 나타내는지 검사

<br>

#### 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞을 때
- 대부분 Set 구현체는 AbstractSet / List 구현체는 AbstractList / Map 구현체는 AbstractMap 의 equals 상속받아 사용

<br>

#### 클래스가 private이거나 package-private이고 equals 메소드를 호출할 일이 없을 때
- equals 메소드가 실수로 호출되는 것을 막고 싶다면

```java
@Override
public boolean equals (Object o){
  	throw new AssertionError();	// 호출 금지
}
```

<br>

## equals 메소드를 재정의해야 할 상황
#### 객체 식별성(object identity: 두 객체가 물리적으로 같은가)가 아니라 논리적 동치성을 확인해야 하는데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때
- 주로 값 클래스가 해당 : Integer와 String처럼 값을 표현하는 클래스를 말함
- 값을 비교하길 원하는 프로그래머의 기대에 부흥하고 Map의 키, Set의 원소로 사용할 수 있게 된다. 

<br>

#### 값 클래스여도 equals를 재정의할 필요없을 때
- 인스턴스 통제 클래스 : 값이 같은 인스턴스가 둘 이상 만들어 지지 않음 (EX. Static Factory Method Pattern, Enum)
- 논리적 동치성과 객체 식별성이 같은 의미가 되고 Object의 equals가 논리적 동치성까지 확인해준다.

<br>

## equals 메서드의 규약
- 동치 관계(equivalence relation)를 구현 : 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산
- 이 부분집합을 동치류(equivalence class, 동치 클래스)라 한다.

### 특징
- 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해 x.equals(y)는 true이다.
> - 객체는 자기 자신과 같아야 한다. 

<br>

- 대칭성(symmetry) : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면 y.equals(x)도 true이다.
> - 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.

```java
    // 대칭성 위배!
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }

    //올바르게 수정, instanceof String 부분 삭제
    @Override public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
```

<br>

- 추이성(transitivity) : null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고 y.equals(z)도 true이면, x.equals(z)도 true이다.
> - 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻이다. 

1 . 대칭성 위배

```java
// ColorPoint의 equals
  @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```

```java
    Point p = new Point(1, 2);
    ColorPoint cp = new ColorPoint(1, 2, Color.RED);
    System.out.println(p.equals(cp) + ", " + cp.equals(p)); //true(Point의 equals), false(ColorPoint의 equals)
```

<br>

2 . 추이성 위배

```java
// ColorPoint의 equals
   @Override public boolean equals(Object o) {
       if (!(o instanceof Point))
           return false;

       // o가 일반 Point면 색상을 무시하고 비교한다.
       if (!(o instanceof ColorPoint))
           return o.equals(this);

       // o가 ColorPoint면 색상까지 비교한다.
       return super.equals(o) && ((ColorPoint) o).color == color;
   }
```

```java
    ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
    Point p2 = new Point(1, 2);
    ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
    System.out.printf("%s , %s , %s%n", p1.equals(p2), p2.equals(p3), p1.equals(p3)); //true(2번째 if문에서 Point의 equals), true(Point의 equals), false(ColorPoint의 equals)
```

<br>

3 . 무한 재귀 문제

```java
//SmellPoint의 equals
@Override public boolean equals(Obejct o){
  if(!(o instanceof Point))
    return false;
  if(!(o instanceof SmellPoint))
    return o.equals(this);
  return super.equals(o) && ((SmellPoint) o).color == color;
}
```

```java
    ColorPoint p1 = new ColorPoint(1,2, Color.RED);
    SmellPoint p2 = new SmellPoint(1,2);
    p1.equals(p2);
    // 처음에 ColorPoint의 equals > 2번째 if문 걸리고 SmellPoint의 equals 비교
    // SmellPoint의 equals > 2번쨰 if문 ColorPoint의 equals
    // 무한 재귀 > StackOverflowError
```

- getClass 검사를 통해 규약도 지키고 값도 추가하면서 구체 클래스를 상속할 수 있다는 것은 아니다. 이는 리스코프 치환 원칙을 위배하기 때문이다. 

```java
@Override public boolean equals(Object o){
  if(o == null || o.getClass() != getClass())
    return false;
  Point p = (Point) o;
  return p.x == x && p.y == y;
}
```

- 리스코프 치환원칙 (Liskov substitution principle) : 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다.
>  - Point의 하위클래스는 정의상 여전히 Point이므로 어디서든 Point로 활용가능해야 한다. 

<br>

### 우회방법 
#### 상속대신 컴포지션을 사용하라
- 컴포지션(composition) : 기존 클래스가 새로운 클래스의 구성요소가 되는 것, 상속의 단점을 커버할 수 있다.

```java
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    /**
     * 이 ColorPoint의 Point 뷰를 반환한다.
     */
    public Point asPoint() {
        return point;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }

    @Override public int hashCode() {
        return 31 * point.hashCode() + color.hashCode();
    }
}
```

<br>

#### 추상 클래스의 하위 클래스 사용하기
- 추상 클래스의 하위 클래스에서라면 equals 규약을 지키면서 값을 추가할 수 있다.
- 상위 클래스를 직접 인스턴스화하는 것이 불가능하기 때문에 하위 클래스끼리 비교가 가능해진다. 

<br>

- 일관성(consistency) : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
> - 두 객체가 같다면 앞으로도 영원히 같아야 한다는 뜻이다.
> - 클래스를 만들 때는 불변 클래스로 만드는 게 나을지 심사숙고하자(ITEM17)
> - 불변 클래스로 만들기로 했다면 한 번 같은 것은 계속 같고, 한 번 다른 것은 계속 달라야 한다. 

- null-아님 : null이 아닌 모든 참조값 x에 대해, x.equals(null)은 false이다.
> - 모든 객체가 null과 같지 않아야 한다는 뜻이다. 

1 . 명시적 null 검사

```java
@Override public boolean equals(Object o){
  if( o == null){
    return false;
  }
}
```

<br>

2 . 묵시적 null 검사

```java
@Override public boolean equals(Obejct o){
  if(!(o instanceof MyType)) 	
    return false;
  MyType mt = (MyType) o;
}
```

- instanceof 자체가 타입과 무관하게 null이면 false 반환

<br>

## 양질의 equals 메소드 구현하기

1 . == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다. 

2 . instanceof 연산자로 입력이 올바른 타입인지 확인한다.
- 올바른 타입은 equals 가 정의된 클래스인 것이 보통이다.
- 올바른 타입이 아닌 경우(구현한 특정 인터페이스) 구현한 클래스 간 비교가 가능하게 해야한다. 이 때는 인터페이스의 equals 를 이용하여 비교해야 한다.
> - EX) Set, List, Map, Map.Entry 등의 컬렉션 인터페이스

3 . 입력을 올바른 타입으로 형변환한다.
- 2번에서 검사했기 때문에 100% 통과

4 . 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다. 
- 2번에서 인터페이스를 사용했다면 입력 필드값을 가져올 때도 해당 인터페이스의 메소드를 사용해야 한다. 

<br>

## equals 메소드 구현 추가 주의 사항

### 기본 타입, 참조 타입

- float, double 을 제외한 기본 타입 :  == 연산자
- 참조 타입 :  equals 메소드 비교
- float, double :  Float.compare(float, float), Double.compare(double, double) 부동 소수값 비교 
> - Float.equals(float)나 Double.equals(double) 은 오토 박싱을 수반할 수 있어 성능상 좋지 않다.

- 배열 필드 : 원소 각각을 지침대로 비교
> - 모두가 핵심 필드라면 Arrays.equals() 메소드 사용

<br>

### null 정상값 취급을 방지하자
- 정적 메소드인 Object.equals(object, object) 로 비교하여 NullPointerException 발생을 예방한다.

<br>

### 필드의 표준형을 저장하여 이용하자
- 비교하기 복잡한 필드를 가진 클래스는 필드의 표준형(canonical form) 을 저장해 비교하면 경제적이다.
> - 불변 클래스에 제격

<br>

### 필드 비교 순서는 equals 의 성능을 좌우한다
- 다를 가능성이 크거나 비교 비용이 싼 필드 먼저
- 핵심 필드와 파생 필드를 구분하여 비교하자

<br>

### equals를 재정의할 때는 hashCode를 반드시 재정의하자(ITEM11)

<br>

### 너무 복잡하게 해결하려 들지 말자

<br>

### Object 외의 타입을 매개변수로 받는 equals 메소드는 선언하지 말자

```java
public boolean equals(MyClass o){
    //... 잘못된 예, 입력 타입은 반드시 Object
}
```

- 재정의가 아니라 다중정의한 것이다. 
- @Override 어노테이션을 잘쓰면 오류를 예방할 수 있다. 

<br>

### AutoValue 프레임워크로 테스트할 수 있다
- 클래스에 해당 어노테이션 `@AutoValue` 만 추가하면 알아서 작성해준다. 
- [https://www.baeldung.com/introduction-to-autovalue](https://www.baeldung.com/introduction-to-autovalue)

<br>

## 결론
- 꼭 필요한 경우가 아니면 equals를 재정의하지 말자
- 많은 경우 Object의 equals가 대부분 원하는 비교를 정확히 수행해준다.
- 재정의하는 경우 그 클래스이 핵심 필드를 빠짐없이 다섯 가지 규약을 지켜가며 비교해야 한다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)