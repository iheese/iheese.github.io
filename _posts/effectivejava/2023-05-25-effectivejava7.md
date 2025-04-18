---
layout: post
title: '[EFFECTIVE JAVA] ITEM 7, 다 쓴 객체 참조를 해제하라'
subtitle: ''
date: 2023-05-25 15:40:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 7, 다 쓴 객체 참조를 해제하라

<br>

## GC의 메모리 누수

```java
// 코드 7-1 메모리 누수가 일어나는 위치는 어디인가? (36쪽)
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
```

- 메모리 누수 : 가비지 컬렉션 활동과 메모리 사용량이 늘어나 성능이 저하된다. 
- 위 스택이 줄어들 때 꺼내진 객체들은 GC가 회수해가지 않는다. 스택에 여전히 해당 객체들의 참조를 가지고 있기 때문이다. 

#### 해결방법
- 해당 객체의 참조를 다 썼을 때 null 처리(참조 해제) 해주면 된다. 

```java
//    // 코드 7-2 제대로 구현한 pop 메서드 (37쪽)
//    public Object pop() {
//        if (size == 0)
//            throw new EmptyStackException();
//        Object result = elements[--size];
//        elements[size] = null; // 다 쓴 참조 해제
//        return result;
//    }

    public static void main(String[] args) {
        Stack stack = new Stack();
        for (String arg : args)
            stack.push(arg);

        while (true)
            System.err.println(stack.pop());
    }
}
```

<br>

## 객체 참조를 null 처리하는 것은 예외적인 경우이다
- 객체 참조를 null 처리하는 것은 예외적인 경우여야 한다. 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어낸다.
- 스택이 메모리 누수에 취약한 이유
> - 스택이 자기 메모리를 직접 관리하기 때문이다.
> - 배열의 활성 영역에 속한 원소들이 사용되고 비활성 영역은 쓰이지 않는다.
> >  - 활성 영역 : 인덱스가 size보다 작은 원소들로 구성된다. 그 외는 비활성 영역
> - 문제는 위 사실을 가비지 컬렉터는 알 길이 없다. 그러므로 null 처리로 가비지 컬렉터에게 알려줘야 한다. 

<br>

## 메모리 누수의 주범

### 자기 메모리를 직접 관리하는 클래스

#### 해결책

- 원소를 사용한 즉시 그 원소가 참조한 객체를 null 처리해줘야 한다.

### 캐시

- 객체 참조를 캐시에 넣고 그 객체를 다 쓴 뒤로도 한참을 나둔다.

#### 해결책
- WeakHashMap을 사용해 캐시를 만든다. 
> - 외부에서 키를 참조하는 동안만 엔트리가 살아 있다
- 시간이 지날수록 엔트리의 가치를 떨어뜨는 방식 
> - ScheduledThreadPoolExecutor : 쓰지 않는 엔트리 청소
> - LinkedHashMap은 `removeEldestEntry()` 메소드를 사용하여 엔트리 청소
- 복잡한 캐시는 `java.lang.ref` 참고

### 리스너, 콜백

- 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않으면 계속 쌓인다. 

#### 해결책

- 약한 참조로 저장, WeakHashMap에 키로 저장


<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 깃헙](https://github.com/WegraLee/effective-java-3e-source-code)