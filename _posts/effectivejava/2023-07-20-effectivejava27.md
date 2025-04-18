---
layout: post
title: '[EFFECTIVE JAVA] ITEM 27, 비검사 경고를 제거하라'
subtitle: ''
date: 2023-07-20 11:00:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 27, 비검사 경고를 제거하라

<br>

## 제네릭을 사용하면 나게 될 컴파일러 경고
- 비검사 형변환 경고
- 비검사 메소드 호출 경고
- 비검사 매개변수화 가변인수 타입 경고
- 비검사 변환 경고

<br>


## 다이아몬드 연산자 해결
- 자바7 부터 제네릭 타입 추론이 가능해졌다. 

```java
Set<Lark> exaltation = new HashSet();
```

- 잘못된 코드
- javac 명령줄에 -Xlint:uncheck 옵션을 추가하면 컴파일러가 해결법을 알려준다. 
- 해결법 : 타입 매개변수를 명시해주거나, 다이아몬드 연산자로 해결할 수 있다.

```java
Set<Lark> exaltation = new HashSet<>();
```

- 다이아몬드 연산자를 이용하면 컴파일러가 실제 타입 매개변수를 추론한다.

<br>

## 할 수 있는 한 모든 비검사 경고를 제거하라
- 타입 안정성이 보장된다.
- 런타임시 ClassCastException이 발생할 일이 없다.
- 의도한 대로 잘 동작하리라 확신할 수 있다. 

<br>

## 경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있으면 @SuppressWarning("unchecked") 애너테이션을 달아 경고를 숨기자
- 경고를 숨기는 이유 : 검증된 비경고 경고를 숨기지 않으면 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있다.
- 타입 안전함을 검증하지 않은 채 경고를 숨기면 스스로에게 잘못된 보안 인식을 심어주는 꼴이 된다. 
- 경고 없이 컴파일되겠지만 런타임에는 ClassCastException을 던질 수 있다. 

### @SuppressWarnings 애너테이션
- 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다.

<br>

## @SuppressWarnings 애너테이션은 항상 가능한 좁은 범위에 적용하자
- 보통은 변수 선언, 아주 짧은 메소드, 혹은 생성자가 될 것이다.
- 심각한 경고를 놓칠 수 있으니 절대로 클래스 단위에 적용하면 안된다. 

### 한줄이 넘는 메소드나 생성자가 달린 @SuppressWarnings 애너테이션을 발견하면 지역변수 선언 쪽으로 옮기자

- ArrayList의 toArray 메소드

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size) { 
        return (T[]) Arrays.copyOf(elements, size, a.getClass());
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size) {
        a[size] = null;
    }
    return a;
}

```

- 위 코드를 컴파일하면 `return (T[]) Arrays.copyOf(elements, size, a.getClass());` 이쪽 코드에 경고가 생성된다. 
> - required: T[], found: Object[]
- @SuppressWarnings은 선언에만 달 수 있어 return 문에는 달 수 없다. 아래처럼 해결할 수 있다. 


```java
public <T> T[] toArray(T[] a) {
    if (a.length < size) { 

        @SuppressWarnings("unchecked")
        T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size) {
        a[size] = null;
    }
    return a;
}

```

<br>

## @SuppressWarnings("unchecked") 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다
- 다른 사람이 그 코드를 이해하는 데 도움이 된다.
- 다른 사람이 그 코드를 잘못 수정하여 타입 안정성을 잃는 상황을 줄여준다.
- 코드가 안전한 근거가 쉽게 떠오르지 않더라도 끝까지 포기하지 말자, 근거를 찾는 중에 그 코드가 사실은 안전하지 않다는 것을 발견할 수 있다.

<br>

## 정리
- 비검사 경고는 중요하니 무시하면 안된다.
- 모든 비경고 검사는 런타임시 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하니 최선을 다해 제거해야 한다.
- 경고를 없앨 방법을 찾지 못했다면 코드의 타입 안전함을 증명하고 가능한 한 범위를 좁혀 `@SuppressWarnings("unchecked")` 을 사용해 경고를 숨기고 주석을 달아 설명하자.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
 