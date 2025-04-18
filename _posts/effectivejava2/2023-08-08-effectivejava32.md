---
layout: post
title: '[EFFECTIVE JAVA] ITEM 32, 제네릭과 가변 인수를 함께 쓸 때는 신중하라'
subtitle: ''
date: 2023-08-08 12:50:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 32, 제네릭과 가변 인수를 함께 쓸 때는 신중하라

<br>

## 가변인수와 제네릭을 함께 사용할 때의 헛점
> - 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해준다.
- 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다. 
- 내부로 감춰야 했을 해당 배열을 클라이언트에 노출되는 문제가 생긴다.
- 실체화되지 않는 제네릭과 매개변수화 타입을 메소드의 varargs 매개변수를 선언하면 그 호출에 대한 경고를 내보낸다. 
> - 실체화 불가 타입 : 런타임에는 컴파일 타임보다 타입 관련 정보를 적게 담고 있다.


```java
static void dangerous(List<String>...stringLists){
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList;    // 힙오염 발생
  String s = stringLists[0].get(0)     // ClassCastException
}
```

- 매개변수화 타입의 변수가 다른 객체를 참조하면 힙 오염이 발생한다.
- 제네릭과 varargs를 혼용하면 타입 안전성이 깨진다.
- 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

### 그럼에도 제네릭(매개변수화 타입의) varargs 매개변수를 받는 이유
- varargs 매개변수를 받는 메소드가 매우 실무에서 유용하다.
- EX) `Arrays.asList(T...a)`, `Collections.addAll(Collection<? super T> c, T... elements)`, `EnumSet.of(E first, E... rest)`
> - 위 예시는 모두 타입 안전하다.

<br>

## @SafeVarargs 애너테이션
- 전에는 `@SuppressWarnings("unchecked")` 을 사용해 경고를 숨겨야 했다.
- 지루한 작업, 가독성을 떨어뜨리고 떄로는 진짜 문제를 알려주는 경고를 숨기는 안좋은 결과로 이어지기까지 했다.

-  `@SafeVarargs` 애너테이션은 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 해준다.
- 메소드 작성자가 그 메서드가 타입 안전함을 보장하는 장치이다.  메소드가 안전한 것이 확실하지 않으면 절대 @SafeVarargs를 달면 안된다. 

<br>

## 메소드가 안전한지 확신할 수 있을 때
> - 가변인수 메소드를 호출할 때 varargs 매개변수를 담는 제네릭 배열이 만들어진다.
- 메소드가 varargs 매개변수를 담는 배열에 아무것도 저장하지 않을 때
- varargs 배열의 참조가 밖으로 노출되지 않을 때 
- 즉, varargs 매개변수 배열이 호출자로부터 그 메소드로 순수하게 인수들을 전달하는 일만 한다면 안전하다.  

<br>

## 제네릭 varargs 매개변수 배열에 다른 메소드가 접근하는 것을 허용하면 안전하지 않다

#### varargs 매개변수 배열에 아무것도 저장하지 않고 타입 안정성을 깨는 예

```java
static <T> T[] toArray(T... args){
  return args; 
}
```

- 위 예는 가변인수로 넘어온 매개변수를 배열에 담아 반환하는 제네릭 메소드이다. 
- 위에서 생성되는 배열은 Object[] 이다.

```java
static <T> T[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
        case 0: return toArray(a, b);
        case 1: return toArray(a, c);
        case 2: return toArray(b, c);
    }
    throw new AssertionError(); // 도달할 수 없다.
}

public static void main(String[] args) {
    String[] attributes = pickTwo("좋은", "빠른", "저렴한");
    System.out.println(Arrays.toString(attributes));
}
```

- 이 위에서는 Object[] > String[] 으로 형변환하는 코드를 컴파일러가 자동 생성한다.
> -  Object[]는 String의 하위 타입이 아니므로 형변환을 실패한다. 
- 자신의 varargs 매개변수 배열을 그대로 반환하면 힙 오염을 이 메소드를 호출한 쪽의 콜스택까지 전이하는 결과를 낳을 수 있다.

### 예외 사항
- @SafeVarargs 로 제대로 애노테이트된 또 다른 varargs 메소드로 넘는 것은 안전하다.
- 이 배열 내용의 일부 함수를 호출만 하는 (varargs를 받지 않는) 일반 메소드에 넘기는 것도 안전하다. 

<br>

### 제네릭 varargs 매개변수를 안전하게 사용하는 전형적인 예

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
```

- 제네릭, 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달자
- 그리고 모든 varargs 메소드를 안전한지 확인하고 아래 두 조건을 어겼다면 수정해라
> - varargs 매개변수 배열에 아무것도 저장하지 않는다.
> - 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다. 

<br>

## varargs 매개변수를 List 매개변수로 바꾸기(ITEM 28)

```java
static <T> List<T> flatten(List<List<? extends T>> lists){
  List<T> restul = new ArrayList<>();
  for(List<? extends T> list : lists)
    result.addAll(list);
  return result;
}
```

- 컴파일러가 이 메소드의 타입 안전성을 검증할 수 있다.
- @SafeVarargs 를 달지 않아도 된다.
- 실수로 안전하다고 판단할 걱정이 없다.
- toArray처럼 varargs 메소드를 안전하게 작성하는 게 불가능한 상황에서도 쓸 수 있다. 

```java
static <T> List<T> pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return List.of(a, b);
            case 1: return List.of(a, c);
            case 2: return List.of(b, c);
        }
        throw new AssertionError();
    }

public static void main(String[] args) {
    List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
    System.out.println(attributes);
}
```

<br>

## 정리
- 가변인수와 제네릭은 궁합이 좋지 않다.
- 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고 배열, 제네릭의 타입 규칙이 다르기 때문이다.
- 제네릭 varargs 매개변수를 타입 안전하지 않지만 허용된다. 
- 메소드에 제네릭(매개변수화된) varargs 매개변수를 사용하고자 한다면 먼저 그 메소드가 타입 안전한지 확인한 다음 @SafeVarargs 애너테이션을 달아 사용하는데 불편함이 없게 하자. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
- [[Java] 제네릭과 와일드카드 타입에 대해 쉽고 완벽하게 이해하기(공변과 불공변, 상한 타입과 하한 타입) _ 망나니개발자](https://mangkyu.tistory.com/241)