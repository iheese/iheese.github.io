---
layout: post
title: '[EFFECTIVE JAVA] ITEM 29, 이왕이면 제네릭 타입으로 만들라'
subtitle: ''
date: 2023-07-27 15:00:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 29, 이왕이면 제네릭 타입으로 만들라

<br>

## 일반 클래스를 제네릭 클래스로 만드는 방법

<br>

### 1. 클래스 선언에 타입 매개 변수를 추가한다.
- 보통 `E` 를 사용한다.
> - Object -> E

```java
public class Stack<E> {
  private E[] elements;
  ...
}
``` 

<br>

### 2. E와 같은 실체화 불가 타입으로는 배열을 만들 수 없으니 해결한다.
- 런타임시 타입 정보가 소거된다. 

```java
E[] elements = new E[DEFAULT_INITAL_CAPACITY]
```

<br>

#### 2(1). 제네릭 배열 생성을 금지하는 제약을 대놓고 우회한다.

- Object 배열을 생성한 다음 제네릭 배열로 형변환한다.

```java
E[] elements = (E[]) new Object[DEFAULT_INITAL_CAPACITY]
```

- 비검사 형변환이 프로그램의 타입 안정성을 해치지 않음을 우리 스스로 확인해야 한다.
> - elements 배열은 private 에 저장된다.
> - 클라이언트로 반환되거나 다른 메서드로 전달되는 일이 없다.
> - 배열에 저장되는 원소의 타입은 항상 E 이다. 
- 검증 되었다면 @SuppressWarning 애너테이션으로 해당 경고를 숨긴다.(ITEM 27)

#### 2(1). 특징
- 가독성이 좋다.
> - 배열의 타입을 E[]로 선언하여 오직 E 타입 인스턴스만 받음을 확실히 어필한다.
- 코드가 더 짧다.
> - 형변환을 배열 생성 시 한 번만 해주면 된다.
- 2(2) 방법보다 더 선호되지만, 배열의 런터임 타입이 컴파일타임 타입과 달라 힙 오염이 일어날 수 있다. 
> - 힙 오염(Heap pollution): JVM의 힙(Heap) 메모리 영역에 저장되어있는 특정 변수(객체)가 불량 데이터를 참조함으로써, 만일 힙에서 데이터를 가져오려고 할때 얘기치 못한 런타임 에러가 발생할 수 있는 오염 상태를 일컫는다.

<br>

### 2(2). 배열 필드의 타입을 Object[]로 바꾼다. 

```java
E result = (E) elements[--size];
```

- E 는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한 지 증명할 방법이 없다.
- 우리가 직접 증명하고 경고를 숨길 수 있다.
> - 해당 할당문을 포함한 메소드 전체의 경고를 숨기지 말고 비검사 형변환을 수행하는 할당문에서만 숨긴다.

```java
@SuppressWarning("unchecked") E result = (E) elements[--size];
```

#### 2(2). 특징
- 배열에서 원소를 읽을 때마다 해줘야 한다. 
- 힙 오염이 없다.
> - 이 때문에 2(2)를 고수하기도 한다.

<br>

## 제네릭 배열을 사용하는 이유
- 'ITEM 28, 배열보다는 리스르틀 우선하라' 와 모순된 점이 있다.
- 제네릭 타입안에서 리스트를 항상 가능한 것도, 더 좋은 것도 아니다.
- 자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList 같은 제네릭 타입도 기본 타입인 배열을 통해 구현해야 한다.
- 성능 향상의 이슈로 배열을 사용할 수 있다.
> - HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용한다.
- 제네릭 타입의 타입 매개변수에는 기본 타입을 사용할 수 없으나, 박싱된 기본 타입으로 우회할 수 있다.

```java
O : Stack<Object>, Stack<int[]>, Stack<List<String>>, Stack
X : Stack<int>, Stack<double> // 컴파일 에러 int > Integer, double > Double
```

<br>

## 한정적 타입 매개변수

```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```

- `<E extends Delayed>` 는 Delayed 의 하위 타입만 받는다는 뜻이다.
> - 모든 타입은 자기 자신의 하위 타입, `DelayQueue<Delayed>` : 사용가능
- ClassCastException 걱정할 필요가 없다. 

<br>

## 정리
- 클라이언트에서 직접 형변환하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.
- 새로운 타입을 설꼐할 때는 형변환 없이도 사용할 수 있게 해야하고 그럴려면 제네릭을 사용해야 할 경우가 많다.
- 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하는 게 좋다.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
 