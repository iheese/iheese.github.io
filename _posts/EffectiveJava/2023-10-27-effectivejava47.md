---
layout: post
title: '[EFFECTIVE JAVA] ITEM 47, 반환 타입으로는 스트림보다 컬렉션이 낫다'
subtitle: ''
date: 2023-10-27 13:52:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 47, 반환 타입으로는 스트림보다 컬렉션이 낫다

<br>

## 원소 시퀀스(일련의 원소)를 반환하는 메소드의 반환 타입
- Collection, Set, List 같은 컬렉션 인터페이스
- Iterable
- Array
> - 기본은 컬렉션이었고, 일부 Collection 메소드를 구현할 수 없을 때는 Iterable 인터페이스, 원소들이 기본타입, 성능에 민감하면 배열을 썼다.

<br>

## Stream 의 등장
> - Java 8 부터 등장
- 스트림은 반복(iteration)을 지원하지 않는다.
- 따라서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다. 

## Stream을 for-each로 반복하기
- Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메소드를 포함할 뿐만 아니라, Iterable 인터페이스가 정의한 방식대로 움직인다.
- 그럼에도 Stream을 for-each 로 반복할 수 없는 이유는 Stream이 Iterable을 확장(extend)하지 않았기 때문이다.

```java
// for-each 예시
for (String s : stringArrayList) {
    System.out.println(s);
}
```

<hr>

```java
for(ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
    // ...프로세스
}
```

- Stream의 iterator 메소드에 메소드 참조를 건네는 방식은 Iterable 로 적절히 형변환을 해줘야 한다.
- 위 방식은 매우 난잡하고 직관성이 떨어진다.

<br>

### Adapter 도입
- 해당 문제를 해결하기 위해서 어댑터를 도입한다. 

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) { 
        return stream::iterator;
}  
```

- 자바는 제공하지 않지만, 위처럼 Stream<E>를 Iterable<E> 로 중개해주는 어댑터를 만들 수 있다.

```java
for(ProcessHandle p : iterableOf(ProcessHandle.allProcesses())){
    //...프로세스
}
```

- 반대로 Iterable<E> 를 Stream<E> 로 중개해주는 어댑터를 만들 수 있다. 이 역시 자바가 제공하진 않는다. 

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
       return StreamSupport.stream(iterable.spliterator(), false);
}
```

<br>

## Collection
- 객체 시퀀스를 반환하는 메소드를 작성하는데, 이 메소드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면 스트림을 반환해도 된다.
- 반대로 반환된 객체들이 반복문에서만 쓰일 걸 안다면 Iterable을 반환하면 된다. 
- 공개 API를 작성할 때는 위 두 상황 모두 배려해야 한다.

<br>

# 원소 시퀀스를 반환하는 공개 API의 타입에는 Collection이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.
- Collection 인터페이스는 Iterable의 하위 타입이고, stream 메소드도 제공하니, 반복, 스트림을 동시에 지원하기 때문이다.
- Arrays 역시 Arrays.asList 와 Stream.of 메소드로 반복과 스트림을 지원할 수 있다.

## 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다.
- 반복되는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList, HashSet 같은 표준 컬렉션 구현체를 반환하는 게 최선일 수 있다.

### 반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 것을 검토해보자
- 아래는 주어진 집합의 멱집합을 반환하는 상황
> - 멱집합 : 한 집합의 모든 부분집합, 원소가 n개면 멱집합의 원소 갯수는 2^n 개 이다. 


```java
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException(
                "집합에 원소가 너무 많습니다(최대 30개).: " + s);
                
        return new AbstractList<Set<E>>() {
            @Override public int size() {
                // 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱 것과 같다.
                return 1 << src.size();
            }

            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

			// 인덱스 n 번째 비트 값 : 해당 원소가 원래 집합의 n 번째 원소를 포함하는지 여부
            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }
}
```

- AbstractList를 이용해 훌륭한 전용 컬렉션을 구현하였다. 
> - 각 원소의 인덱스를 비트 벡터로 사용했다. 
- AbstractCollection을 활용해서 Collection 구현체를 작성할 때는 Iterable 용 메소드 외에 contains, size 메소드를 구현하면 된다. 
> - 시퀀스의 내용을 확정하지 못한다는 등의 사유로 두 메소드를 구현하는 것이 불가능하다면 스트림이나 Iterable을 리턴하는 편이 낫다.

```java
public class SubLists {
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()),
                prefixes(list).flatMap(SubLists::suffixes));
    }

    private static <E> Stream<List<E>> prefixes(List<E> list) { // (a), (a,b), (a,b,c)
        return IntStream.rangeClosed(1, list.size()) // 종료값 포함
                .mapToObj(end -> list.subList(0, end));
    }

    private static <E> Stream<List<E>> suffixes(List<E> list) { // (a,b,c), (b,c), (c)
        return IntStream.range(0, list.size()) //종료값 불포함
                .mapToObj(start -> list.subList(start, list.size()));
    }
}
```

- 입력 리스트의 모든 부분리스트룰 스트림으로 반환한다.
- 어떤 리스트의 부분 리스트는 단순히 프리픽스의 서픽스/ 서픽스의 프리픽스에 빈 리스트 하나만 추가하면 되게 된다. 
- Stream.concat 메소드로 빈리스트를 추가하고, flatMap 메소드로 모든 프리픽스의 모든 서픽스로 구성된 하나의 스트림을 만든다.
> - 참고 : 위 예시 메소드는 모든 부분 리스트를 반환하진 않는 듯하다. {a, c}가 나오지 않는다. 

<br>

### 참고 사항
- 앞서 본 어댑터는 클라이언트 코드를 어수선하게 만들고 느리게 만든다.
- 전용 컬렉션을 사용하는 방식으로는 코드가 지저분해지지만 스트림을 활용한 구현보다 빨랐다. 

<br>

## 정리
- 원소 스퀀스를 반환하는 메소드를 작성할 때는 스트림, 반복 두 가지 방식으로 처리하길 원하는 사용자가 있음을 떠올리자.
- 컬렉션을 반환할 수 있다면 반환하자.
> - 원소 개수가 적다면 표준 컬렉션을 이용하고, 작지 않다면 전용 컬렉션을 구현할 지 고민해보자.
- 컬렉션을 반환하는 게 불가능하면 스트림과 Iterable 중 자연스러운 것을 반환하자.
- 나중에 Stream 인터페이스가 Iterable을 지원하도록 자바가 수정되다면 안심하고 스트림을 반환하자.
> - 스트림과 반복 모두에 사용될 수 있으니.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
- [Short-Circuit img](https://www.logicbig.com/tutorials/core-java-tutorial/java-util-stream/short-circuiting.html)
