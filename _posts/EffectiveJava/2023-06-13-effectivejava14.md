---
layout: post
title: '[EFFECTIVE JAVA] ITEM 14, Comparable을 구현할지 고려하라'
subtitle: ''
date: 2023-06-13 12:00:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 14, Comparable을 구현할지 고려하라

<br>

## compareTo() 메소드와 equals() 메소드의 차이
- Comparable 인터페이스의 하나 뿐인 compareTo 메소드
- equals와 달리 compareTo()는 Object의 메소드가 아니다.
- compareTo()는 단순 동치성 비교에 더해 순서까지 비교할 수 있고 제네릭하다.
- Comparable을 구현했다는 것은 그 클래스의 인스턴스 간의 자연적인 순서가 있다는 것을 뜻한다. 
- EX) Comparable을 구현한 객체들의 배열은 손쉽게 정렬 가능, `Arrays.sort(a);` 

```java
public interface Comparable<T> {
  int compareTo(T t);
}
```

- 사실상 자바 플랫폼 라이브러리의 모든 값 클래스와 열거타입이 Comparable을 구현했다.
- 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성했다면 Comparable 인터페이스를 구현하자

<br>

## compareTo() 메소드 일반 규약
- equals와 비슷한 내용이다. 

```java
this < object : -1
this == object : 0
this > object : 1
//비교할 수 없는 타입일 경우 ClassCastException
```

### 대칭성

`sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`

- x.compareTo(y)는 y.compareTo(x)가 예외를 던질 때 한해 예외를 던진다.
- 두 참조한 객체 순서를 바꾸어 비교해도 예상한 결과가 나와야 한다.

### 추이성

`x.compareTo(y) > 0 && y.compareTo(z) > 0 이면 x.compareTo(z) > 0 이다.`

- 첫 번쨰가 두 번째보다 크고 두 번째가 세 번째보다 크면 첫 번째는 세 번째보다는 커야한다.

### 반사성

Comparable을 구현한 클래스는 모든 z에 대해 `x.compareTo(y) == 0 이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z)) 다.`

- 크기가 같은 객체들끼리 어떤 객체와 비교해도 같아야 한다. 


#### equals

`(x.compareTo(y) == 0) == (x.equals(y))`

- 필수는 아니지만 지키는 게 좋다. 
- 이를 지키지 않으면 아래와 같은 사실을 명시하는 것이 좋다.
- "주의: 이 클래스의 순서는 equals 메소드와 일관되지 않다."

- equals() 를 제외한 세 규약은 compareTo() 메소드로 수행하는 동치성 검사도 equals() 규약과 같이 충족해야 함을 뜻한다.

<br>

### 주의 사항
- 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 방법이 없다. 
- 우회법 : Comparable을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다면 
> > - 독립된 클래스를 만들고 클래스의 인스턴스를 가리키는 필드를(컴포지션, Composition) 두고 해당 필드를 제공하는 '뷰' 메소드 제공

<br>

## compareTo() 메소드 작성 요령
- equals() 와 비슷하나 몇 가지 차이점에 주의하면 된다.

#### Comparable은 타입을 인수로 받는 제네릭 인터페이스이다. 
- 메소드의 인수 타입은 컴타일 타입에 정해진다. 
- 입력 인수 타입을 확인하거나 형변환할 필요가 없다. 
- 인수 타입이 잘못되면 컴파일이 안된다. 

<br>

#### compareTo 메소드는 필드의 동치가 아니라 순서를 비교한다.
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 Comparator(비교자, 직접 만들거나 자바가 제공)를 대신 사용한다.

<br>

#### compareTo 메소드 구현 시 관계 연산자 <, > 을 사용하는 방식은 오류를 유발하기 쉽다.
- 자바 7부터 적용
- 박싱된 기본 타입 클래스에서는 compare 메소드를 사용하자

```java
Integer.compare(x,y);
Float.compare(x,y);
```

<br>

#### 클래스의 핵심 필드가 여러 개라면 어떤 것을 먼저 비교할 지 고려하고 결과를 곧장 리턴해라 

<br>

#### 비교자 생성 메소드(comparator construction method)와 팀을 꾸려 메소드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.
- 자바 8부터 적용

```java
private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
  }
```

- (PhoneNumber pn) 는 이 상황에서는 자바의 타입 추론 능력이 추론하기 어려워 컴파일되도록 도와준 것이다. 

<br>

#### Comparator의 수많은 보조 생성 메소드 
- `comparingLong.thenComparingLong`, `comparingDouble.thenComparingDouble`
- short는 int용 버전, float는 double용 버전 사용

<br>

#### 값의 차를 이용한 compareTo, compare 메소드를 사용하지 말자
- 정수 오버플로 가능성, 부동소수점 계산 방식 오류 가능성이 존재한다.
- 수정전 : `o1.hashCode() - o2.hashCode();`
- 수정후 : 정적 compare 메소드 `Integer.compare(x,y);` , 비교자 생성 메소드 `Comparator.comparingInt(o -> o.hashCode());`

<br>

## 정리
- 순서를 고려해야 하는 값 클래스를 작성한다면 Comparable 인터페이스를 구현하자.
- 해당 인스턴스들을 쉽게 정렬, 검색, 비교 기능을 제공할 수 있게 하는 컬렉션과 어우러지게 사용하자. 
- compareTo 메소드에서 필드 값 비교할 때는 <와 > 연산자는 쓰지 말아야 하고 박싱된 기본 타입 클래스가 제공하는 정적 compare 메소드, Comparator 인터페이스가 제공하는 비교자 생성 메소드를 사용하자.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
