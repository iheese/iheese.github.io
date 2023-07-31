---
layout: post
title: '[EFFECTIVE JAVA] ITEM 30, 이왕이면 제네릭 메소드로 만들라'
subtitle: ''
date: 2023-07-27 12:00:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 30, 이왕이면 제네릭 메소드로 만들라

<br>

- 클래스와 마찬가지로 메소드도 제네릭으로 만들 수 있다.
- 매개변수화 타입을 받는 정적 유틸리티 메소드는 보통 제네릭이다.
> - EX) Collections의 알고리즘 메소드(binarySearch, sort)

<br>

## 제네릭 메소드 만들기
 
1. 메소드 선언에서 원소 타입(입력, 반환)을 타입 매개변수로 명시한다.
2. 메서드 안에서도 타입 매개변수를 사용하게 수정한다.
3. 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다. 
4. 한정적 와일드카드 타입을 사용하면 반환타입, 입력타입을 더 유연하게 개선할 수 있다.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }
```

<br>
 
## 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때

- 제네릭은 런타임 시 타입 정보가 소거되므로 하나의 객체를 어떤 타입이든 매개변수화 할 수 있다.
- 요청 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 `제네릭 싱글턴 팩터리` 를 만들어야 한다. 
> - EX) Collections.reverseOrder, Collections.emptySet

```java
// 제네릭 싱글턴 팩터리 패턴
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```

- 항등함수를 담은 클래스를 만들려고 할 때, 제네릭이 실체화되지 않아(소거 방식) 항등함수를 타입별로 하나씩 만들 필요가 없어졌다.
> - 항등함수 : 입력 값을 수정 없이 그대로 반환하는 특별한 함수
- 타입 안전하다는 사실을 알고 있으니 @SuppressWarnings 어노테이션으로 비검사 형변환 경고를 숨겨준다.

<br>

### 예시

```java
    public static void main(String[] args) {
        String[] strings = { "삼베", "대마", "나일론" };
        UnaryOperator<String> sameString = identityFunction();
        for (String s : strings)
            System.out.println(sameString.apply(s));

        Number[] numbers = { 1, 2.0, 3L };
        UnaryOperator<Number> sameNumber = identityFunction();
        for (Number n : numbers)
            System.out.println(sameNumber.apply(n));
    }
```

- 제네릭 싱글턴을 다른 타입으로 사용하는 모습이다. 

<br>

### 재귀적 타입 한정

- 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다. 
- 주로 타입의 자연적 순서를 정하는 Comarable 인터페이스와 함께 쓰인다. 

```java
public interface Comparable<T> {
  int compareTo(T o);
}
```

- T는 Comparable<T> 를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다.
- 모든 타입이 자신과 가은 타입의 원소와만 비교할 수 있다. 

```java
// 최댓값 계산
   public static <E extends Comparable<E>> E max(Collection<E> c) {
        if (c.isEmpty())
            throw new IllegalArgumentException("컬렉션이 비어 있습니다.");

        E result = null;
        for (E e : c)
            if (result == null || e.compareTo(result) > 0)
                result = Objects.requireNonNull(e);

        return result;
    }
```

- <E extends Comparable<E> == "모든 타입 E는 자신과 비교할 수 있다.(상호 비교 가능)"

<br>

## 정리

- 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메소드보다 제네릭 메소드가 더 안전하다.
- 메소드도 형변환 없이 사용할 수 있는 편이 좋고 그럴려면 제네릭 메소드가 되어야 한다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
 