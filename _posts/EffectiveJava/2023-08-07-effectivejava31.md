---
layout: post
title: '[EFFECTIVE JAVA] ITEM 31, 한정적 와일드카드를 사용해 API 유연성을 높이라'
subtitle: ''
date: 2023-08-07 13:00:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 31, 한정적 와일드카드를 사용해 API 유연성을 높이라

<br>

## 매개변수화 타입은 불공변이다
- 서로 다른 타입 Type1, Type2가 있을 때 List<Type1>은 List<Type2> 의 하위 타입도 상위 타입도 아니다.
- List<String>은 List<Object>의 하위 타입이 아니다.
> -  List<Object> 에는 어느 객체든 넣을 수 있지만 List<String>에는 문자열만 넣을 수 있다. List<String>은  List<Object>가 하는 일을 제대로 수행하지 못하니 하위 타입이 될 수 없다.
> - 이는 리스코프 치환 원칙에 어긋난다.  
> > - 리스코프 치환 원칙 :  부모 객체와 이를 상속한 자식 객체가 있을 때 부모 객체를 호출하는 동작에서 자식 객체가 부모 객체를 완전히 대체할 수 있다.

<br>

#### 공변/불공변

- 공변(covariant) : A가 B의 하위 타입일 때, T <A> 가 T<B>의 하위 타입이면 T는 공변 , EX) 배열
- 불공변(invariant) : A가 B의 하위 타입일 때, T <A> 가 T<B>의 하위 타입이 아니면 T는 불공변 , EX) 제네릭

<br>

## 한정적 와일드카드 타입

```java
   // 와일드카드 타입을 사용하지 않은 pushAll 메서드 - 결함
   public void pushAll(Iterable<E> src) {
       for (E e : src)
           push(e);
   }

    // 생산자(producer) 매개변수에 와일드카드 타입 적용 
    public void pushAll(Iterable<? extends E> src) {
        for (E e : src)
            push(e);
    }
```

- pushAll 의 입력 매개변수 타입은 'E의 Iterable'이 아니라 "E의 하위 타입이 Iterable"이다.

```java
   // 와일드카드 타입을 사용하지 않은 popAll 메서드 - 결함
   public void popAll(Collection<E> dst) {
       while (!isEmpty())
           dst.add(pop());
   }

    // E 소비자(consumer) 매개변수에 와일드카드 타입 적용 
    public void popAll(Collection<? super E> dst) {
        while (!isEmpty())
            dst.add(pop());
    }
```

- pollALL 의 입력 매개변수 타입이 'E의 Collection'이 아니라 "E의 상위 타입의 Collection"이어야 한다. 

- 유연성을 극대화하려면 원소의 생성자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.
- 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없다. 

### PECS 
- producer-extends, consumer-super
> - 겟풋 원칙 : Get and Put Principle
- 매개변수화 타입 T가 생산자 :  <? extends T> 
- 매개변수화 타입 T가 소비자 :  <? super T>

<br>

## 반환 타입에서의 한정적 와일드카드 타입
- 반환 타입에서는 한정적 와일드카드 타입을 사용하면 안된다.
- 유연성을 높여주지 않고 클라이언트 코드에서도 와일드 카드 타입을 사용하게 하기 때문이다.

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

- 클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API 에 무슨 문제가 있을 가능성이 크다. 

<br>

### 매개변수와 인수

- 매개변수(Parameter) : 메소드 선언에 정의한 변수
> - void add(int value){...}
- 인수(Argument) : 메소드 호출 시 넘기는 실젯값
> - add(10);

- 타입 매개변수(Type Parameter) : 아래에서는 T
> - class Set<T> {...}
- 타입 인수(Type Argument) : 아래에서 Integer
> - Set<Integer> = ...

<br>

## 예시

```java
public static <E extends Comparable<E>> E max(List<E> list)
```

- 위는 아래 리스트를 max 처리할 수 없다. 


```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

- 위처럼 구현해야 아래 리스트를 max 처리할 수 있다.

```java
List<ScheduledFuture<?>> scheduledFutures = ... ;
```

```java
public interface Comparable<E>  
public interface Delayed extends Comparable<Delayed>
public interface ScheduledFuture<V> extends Delayed, Future<V>
```

- 위의 차이의 이유는 위 관계로 선언되어 있기 때문이다.
- Comparable을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드카드가 필요하다. 

![스크린샷 2023-08-07 오후 12 39 51](https://github.com/iheese/TIL/assets/88040158/d2b95555-16e1-427b-9f00-48f28ae2d382)

<br>

## 메소드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라
- 비한정적 타입 매개변수 > 비한정적 와일드카드 
> - `List<E>` > `List<?>`
- 한정적 타입 매개변수 > 한정적 와일드카드
> - `<E extends Number>` > `List<? extends Number>`

<br>

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j); // 더 좋은 방법
```

- 와일드카드 타입을을 사용하면 어떤 리스트든 명시한 인덱스의 원소를 교환해주고, 신경 써야 할 타입 매개변수도 없다.

```java
 public static void swap(List<?> list, int i, int j) {
        list.set(i, list.set(j, list.get(i)));
    }
```

- 하지만 위와 같은 직관적인 코드는 컴파일되지 않는다.
- 리스트 타입이 List<?>인데 List<?>에는 null 외에는 어떤 값도 넣을 수 없다는데 있다.
- 해결법으로는 와일드카드 타입의 실제 타입을 알려주는 private 도우미 메서도로 따로 작성하여 활용하는 방법이 있다.

```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

<br>

## 정리
- 조금 복잡해지지만 와일드카드 타입을 적용하면 API가 유연해진다.
- 널리 쓰일 라이브러리를 작성하면 와일드카드 타입을 적절히 사용하자.
- PECS 공식을 기억하자.
> - producer-extends, consumer-super
- Comparable, Comparator는 모두 소비자이다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
- [[Java] 제네릭과 와일드카드 타입에 대해 쉽고 완벽하게 이해하기(공변과 불공변, 상한 타입과 하한 타입) _ 망나니개발자](https://mangkyu.tistory.com/241)