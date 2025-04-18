---
layout: post
title: '[EFFECTIVE JAVA] ITEM 28, 배열보다는 리스트를 사용하라'
subtitle: ''
date: 2023-07-21 11:40:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 28, 배열보다는 리스트를 사용하라

<br>

## 배열과 제네릭 타입의 차이

### 배열은 공변(covariant)이고, 제네릭은 불공변(invariant)이다
- Sub가 Super의 하위 타입이라면 배열 Sub[]는 배열 Super[]의 하위 타입이 된다.
- 공변, 즉 함께 변한다. 
- 서로 다른 타입 Type1, Type2 가 있을 때 List<Type1>은 List<Type2>의 하위 타입도, 상위 타입도 아니다.

```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException
```

- 배열을 사용하면 런타임 시 실수를 알게 된다.

```java
List<Object> objectList = new ArrayList<Long>(); // 호환 불가
objectList.add("타입이 달라 넣을 수 없다.");
```

- 리스트를 사용하면 컴파일도 되지 않는다. 

<br>

### 배열은 실체화(reify)된다
- 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.
- 반면, 제네릭은 타입 정보가 런타임에는 소거된다. 
> - 즉, 컴파일 단계에서만 원소 타입을 검사하며 런타임에는 알 수 조차 없다. 
> - 소거(erasure)는 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입을 함께 사용할 수 있게 해주는 메커니즘으로 자바5가 제네릭으로 순조롭게 전환될 수 있게 해줬다. 

<br>

## 제네릭 배열은 사용 불가하다
- 베열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.
- EX) new List<E>[], new List<String>[], new E[] : 컴파일 시 제네릭 배열 생성 오류

### 제네릭 배열 사용 불가 이유
- 타입 안전하지 않기 때문이다. 
- 이를 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 ClassCastException이 발생할 수 있다. 

<br>

## 실체화 불가 타임(non-reifiable type)
- 실체화되지 않아서 런타임에는 컴파일 타입보다 타입 정보를 적게 가지는 타입이다. 
- EX) E, List<E>, List<String>
- 소거 메커니즘 때문에 매개변수화 타입 가운데 실체화될 수 있는 타입은 비한정적 와일드카드 타입 뿐이다.
> - EX) List<?>와 Map<?,?> 

<br>

## 배열을 제네릭으로 만들 수 없어 귀찮을 떄

### 제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열을 반환하는 게 불가능하다
- ITEM 33 에서 어느 정도 문제해결 가능 : 타입 안전 이종 컨테이너를 통해 자신의 원소타입 추론

### 제네릭 타입과 가변인수 메소드(varargs method)를 함께 쓰면 해석하기 어려운 경고 메시지를 받게 된다.
- 가변인수 메서드를 호출할 때마다 가변인수 매개변수를 담을 배열이 하나 만들어지는데, 이때 그 배열의 원소가 실체화 불가 타입이면 경고가 발생한다.
- `@SafeVarargs` 애너테이션으로 대처 가능하다.

<br>

## 배열 대신 리스트를 사용하자
- 장점 : 타입 안전성과 상호 운용성이 좋아진다.
- 단점 : 코드가 조금 복잡해지고 성능이 살짝 나빠질 수 있다.

```java
public class Chooser<T> {
    private final Object[] choiceList;

    public Chooser(Collection choices) {
        choiceaArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.[rnd.nextInt(choiceaArray.length)];
    }
}
```

- 위 클래스의 choose 메소드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 한다.
- 혹시 타입이 다른 원소가 들어 있었다면 런타임 시 형변환 오류(ClassCastException) 가 발생할 것이다. 

```java
public class Chooser<T>{
  private final T[] choiceArray;

  public Chooser(Collection<T> choices){
    choicesArra = (T[]) choices.toArray(); 
  }

    public Object choose(){
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```

- 위처럼 제네릭으로 만들기 위해 Object > T[]로 바꿨다.
- T 가 무슨 타입인지 알수 없으니 컴파일러는 형변환이 런타임에서 안전한 것을 보장할 수 없다는 매시지이다.
> - 제네릭에서는 원소의 타입 정보가 소거되어 런타임에는 무슨 타입인지 알 수 없음을 기억해야 한다. 
- 위 프로그램은 작동하긴 하지만 컴파일러가 안전을 보장하지 못한다.
- ITEM 27에 따라 주석을 남기고, 경고를 제거해도 된다.
> - ITEM 27, 비검사 경고를 제거하라

- 하지만 아래 코드처럼 배열 대신 List를 사용하여 경고의 원인을 제거하는 것이 낫다.

```java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

## 정리
- 배열은 공변, 실체화되고, 제네릭은 불공변이고 타입 정보가 소거된다.
- 배열은 컴파일 단계에는 안전하지 않지만 런타임에는 타입 안전하다. 
- 제네릭은 컴파일 단계에는 안전하지만 런타임에는 타입 안전하지 않지 않다. 
- 둘을 섞어쓰는 것은 쉽지 않은 일이며 둘을 섞어 쓰다가 오류, 경고를 발견하면 배열을 리스트로 대체하는 방법을 적용해보는 것이 좋다.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
 