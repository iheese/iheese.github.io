---
layout: post
title: '[EFFECTIVE JAVA] ITEM 26, 로 타입은 사용하지 말라'
subtitle: ''
date: 2023-07-19 11:30:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : false
---

# ITEM 26, 로 타입은 사용하지 말라

<br>

## 제네릭
- 자바5 부터 사용 가능
- 컬렉션이 담을 수 있는 타입을 컴파일러에 알려주는 역할
- 전에는 컬렉션에서 객체를 꺼낼 때마다 형변환이 필요했는데 제네릭을 사용하게 되면서 알아서 형변환 코드를 넣어주고, 엉뚱한 타입의 객체를 넣으려는 시도를 컴파일 단계에서 검증해준다. 
- 안전하고 명확한 프로그램을 만들 수 있다는 장점이 있지만, 더 복잡해진다는 단점이 있다.  

## 제네릭 클래스, 제네릭 인터페이스
- 클래스와 인터페이스 선언에 타입 매개변수가 쓰인 경우
- 두 개를 통틀어 `제네릭 타입` 이라 한다. 
- EX) `List<E>`, E: 원소의 타입을 나타내는 타입 매개변수
- EX) `List<String>` : 원소의 타입이 String인 리스트를 뜻하는 매개변수화 타입

## 로 타입
- 제네릭 타입에서 타입 매개변수를 사용하지 않은 경우
- EX) `List<E>` 의 로 타입은 `List`
- 제네릭 타입 정보가 지워진 것처럼 동작한다.
- 제네릭이 도래하기 전 코드와 호환되도록 하기 위한 궁여지책이다. 

```java
// 컬렉션의 로 타입
private final Collection stamps = ...;
```

```java
// stamp 만 넣어야 하는데 실수로 Coin을 넣는다.
stamps.add(new Coin(...)); // unchecked call 경고
```

```java
// 동전을 꺼내기 전에는 에러를 알 수 없다.
for (Iterator i  = stamps.iterator(); i.hasNext(); ) {
    Stamp stamp = (Stamp) i.next(); // Coin 나오면 ClassCastException을 던진다.
    stamp.cancel();
}
```

#### 오류는 가능한 컴파일 단계에서 발견하는 것이 좋다.

```java
// 매개변수화된 컬렉션 타입
private final Collection<Stamp> stamps = ...;
```

- 타입 안정성을 확보하였다. 

<br>

## 로 타입을 쓰면 제네릭이 안겨주는 안정성과 표현력을 모두 잃게 된다. 

### 그럼 로 타입을 왜 만든건가?
- 호환성 때문이다. 
- 제네릭 없이 짠 코드가 10년 동안 작성되었고 기존 코드를 수용하면서 제네릭을 사용하는 새로운 코드와 맞물려 돌아가게 했어야 했다. 
- 로 타입을 사용하는 메서드에 매개변수화 타입의 인스턴스를 넘겨도 동작해야만 했던 것이다. (그 반대도 마찬가지)
- 마이그레이션 호환성을 위해 로 타입 지원, 제네릭 구현에서는 소거(erasure) 방식 사용
> - 소거(erasure) : 제네릭 타입에 사용된 타입 정보를 컴파일 타입에만 사용하고 런타임에는 소거하는 것을 말한다.

<br>

### List vs List<Object>
- List : 로 타입, 제네릭 타입에서 완전히 발을 뺀 것이다.
- List<Object> : 임의 객체를 허용하는 매개변수화 타입, 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달한 것이다. 
- 제네릭의 하위 타입 규칙 : List<String> 은 로 타입인 List의 하위타입이지만, List<Object>의 하위 타입은 아니다. 
> - List<String>, List<Object> 두 개 다 List 로 타입의 하위 타입

<br>

## 비한정 와일드카드 타입
- 제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않을 때는 물음표(?)를 사용하자
- EX) `Set<E>` 의 비한정 와일드카드 타입은 `Set<?>`

- 로 타입 : 아무 원소나 넣을 수 있으니 타입 불변식을 훼손하기 쉽다.
- 와일드카드 타입 : Collection<?>에는 null 외에는 어떤 원소도 넣을 수 없다.
> - 컬렉션의 타입 불변식을 훼손하지 못하게 막았고, 컬렉션에서 꺼낼 수 잇는 객체의 타입도 알 수 없게 된다. 

<br>

## 로 타입의 규칙 예외

#### class 리터럴에는 로 타입으로 써야 한다. 
- class 리터럴 :  클래스, 인터페이스, 배열 타입들의 이름 또는 기본 타입, `클래스타입.class` 의 형태
- 자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다.
- EX) 허용 : List.class, String[].class, int.class
- EX) 불가 : List<String>.class, List<?>.class

<br>

#### instanceof 연산자는 로 타입을 사용한다. 
- 런타임시 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외에 매개변수화 타입에는 적용할 수 없다. 
- 로 타입이든 비한정적 와일드 카드 타입이든 instanceof는 완전히 똑같이 동작한다. 
> - <?> 는 코드만 지저분하게 만든다. 

```java
if (o instanceof Set) { // 로 타입
    Set<?> s = (Set<?>) o; // 와일드카드 타입 
}

```

<br>

## 용어 정리
- 매개변수화 타입 (parameterized type)
> - EX) List<String>
- 실제 타입 매개변수 (actual type parameter) 
> - EX) String
- 제네릭 타입 (generic type)
> -	List<E>
- 정규 타입 매개변수 (formal type parameter)	
> - EX) E
- 비한정적 와일드카드 타입 (unbounded wildcard type)
> - EX)	List<?>
- 로 타입 (raw type)	
> - EX) List
- 한정적 타입 매개변수 (bounded type parameter)	
> - EX) <E extends Number>
- 재귀적 타입 한정 (recursive type bound)	
> - EX) <T extends Comparable<T>>
- 한정적 와일드카드 타입 (bounded wildcard type)	
> - EX) <? extends Number>
- 제네릭 메서드	(generic method)	
> - EX) static <E> List<E> asList(E[] a)
- 타입 토큰	(type token)
> - EX) String.class

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
 