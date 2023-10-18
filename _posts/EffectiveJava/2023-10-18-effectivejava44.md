---
layout: post
title: '[EFFECTIVE JAVA] ITEM 44, 표준 함수형 인터페이스를 사용하라'
subtitle: ''
date: 2023-10-17 14:50:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 44, 표준 함수형 인터페이스를 사용하라

<br>

## 함수 객체를 매개변수로 받는 생성자와 메소드를 더 많이 만들어야 한다
- 자바가 람다를 지원하면서 API 작성 모범 사례가 바뀌었다.
> - EX) 상위 클래스의 기본 메소드를 재정의해 원하는 동작을 구현하는 템플릿 메소드 패턴의 매력이 줄었다.
- 이를 해결하는 해법은 같은 효과의 함수 객체를 받는 생성자, 메소드를 많이 만들어야 한다.

### 예시 

- LinkedHashMap의 removeEldestEntry 메소드를 아래처럼 재정의하면 최근 원소 100개를 유지하는 캐시처럼 사용할 수 있다.

```java
protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
    return size() > 100;
}
```

- 위에서 size()는 인스턴스 메소드라 가능하다. 
> - 인스턴스가 생성되고 사용되는 메소드라서 가능하다.
- 잘 동작하지만 람다를 사용하면 훨씬 잘 해낼 수 있다. 

```java
@FunctionInterface interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K,V> map, Map.Entry<K, V> eldest);
}
```

- 팩터리나 생성자를 호출할 때는 Map 인스턴스가 존재하지 않아 Map 자신도 넘겨주어야 한다. 

## 필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라
- API가 다루는 개념의 수가 줄어들어 익히기 더 쉬워진다. 
- 표준 함수형 인터페이스들은 유용한 디폴트 메소드를 많이 제공하므로 다른 코드와 상호운용성도 좋아진다. 
> - 위 예에서는 표준 인터페이스인 java.util.function의 BiPredicate<Map<K, V>, Map.Entry<K, V>> 를 사용할 수 있다.  
- java.util.function 패키지 아래 43개 인터페이스가 존재하며 기본적인 6개를 설명하겠다. 

<br>

## 기본 함수형 인터페이스

| 인터페이스 | 함수 시그니처 | 설명 | EX |
|---|---|---|---|
|UnaryOperator<T>|T apply(T t)|반환값과 인수의 타입이 같은 함수, 인수는 1개|String::toLowerCase|
|BinaryOperator<T>|T apply(T t1, T t2)|반환값과 인수의 타입이 같은 함수, 인수는 2개|BigInteger::add |
|Predicate<T>|boolean test(T t)|한 개의 인수를 받아서 boolean을 반환하는 함수|Collection::isEmpty|
|Function<T,R>|R apply(T t)|인수와 반환 타입이 다른 함수|Arrays::asList|
|Supplier<T>|T get()|인수를 받지 않고 값을 반환하는 함수|Instant::now|
|Consumer<T>|void accept(T t)|인수를 하나 받고 반환값이 없는 함수System.out::println|

- [Package java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)
- 위 링크에서 여러 변형을 확인할 수 있다.
- 표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자.
> - 성능 이슈가 생길 수 있다.

<br>

## 표준 함수형 인터페이스를 직접 사용해야 할 때는 언제인가?

### 예시

```java
@FunctionInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}

@FunctionalInterface
public interface ToIntBiFunction<T, U> {
    int applyAsInt(T t, U u);
}
```

- 구조적으로 둘은 동일하다. ToIntBiFunction가 있어도 Comparator가 살아남아야 하는 이유
> - API 에서 굉장히 많이 사용되는데 지금의 이름이 그 용도를 아주 잘 설명해준다.
> - 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있다.
> - 비교자들을 변환하고 조합해주는 유용한 디폴트 메소드들을 많이 가지고 있다. 

- 즉 위 3가지 이유 중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현해야 하는지 고민보는 것이 필요하다. 

<br>

## 직접 만든 함수형 인터페이스에는 @FunctionalInterface 애너테이션을 사용하라
- 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
- 해당 인터페이스가 추상 메소드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
- 그 결과 유지보수 과정에서 누군가 실수로 메소드를 추가하지 못하게 막아준다. 

<br>

## 함수형 인터페이스를 사용할 때 주의점
- 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메소드들을 다중 정의해선 안된다.
> - 이는 클라이언트에게 불필요한 모호함을 남긴다. 

### 예시

```java
public interface ExecutorService extends Executor {
    <T> Future<T> submit(Callback<T> task);
    Future<?> submit(Runnable task);
}
```

- Callable<T>와 Runnable을 각각 인수로 받는 메소드를 다중정의했다.
- 올바른 메소드를 알려주기 위해 형변환을 해야 할 때가 생긴다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
