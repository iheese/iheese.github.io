---
layout: post
title: '[EFFECTIVE JAVA] ITEM 43, 람다보다는 메소드 참조를 사용하라'
subtitle: ''
date: 2023-10-17 11:20:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 43, 람다보다는 메소드 참조를 사용하라

<br>

## 메소드 참조는 함수 객체를 람다보다 더 간결하게 한다

- 람다로 할 수 없는 일이라면 메소드 참조로 할 수 없다.
- 람다로 구현하였을 때 너무 길거나 복잡하면 메소드 참조는 좋은 대안이 되어 준다.
- 람다로 작성할 코드를 새로운 메소드에 담은 다음, 람다 대신 메소드 참조를 사용하는 방식이다.
> - 메소드 참조에는 기능을 잘 드러내는 이름을, 친절한 설명을 문서로 남길 수도 있다.

### 예시

```java
public class Freq {
    public static void main(String[] args) {
        Map<String, Integer> frequencyTable = new TreeMap<>();
        
        for (String s : args) {
            frequencyTable.merge(s, 1, (count, incr) -> count + incr); // 람다
        }
        System.out.println(frequencyTable);
    }
}
```

- merge 메소드는 키, 값, 함수를 인수로 받는다. 
- 위 코드는 키가 맵 안에 없다면 키와 숫자 1을 매핑하고, 이미 있다면 기존 매핑 값을 증가시킨다. 
- 위 코드의 count, incr 는 딱히 하는 일 없이 공간을 차지한다. 이를 개선한다면 아래와 같다.

```java
public class Freq {
    public static void main(String[] args) {
        Map<String, Integer> frequencyTable = new TreeMap<>();
        
        for (String s : args) {
            frequencyTable.merge(s, 1, Integer::sum); // 메서드 참조
        }
        System.out.println(frequencyTable);
    }
}
```

- 매개변수 수가 늘어날 수록 메소드 참조로 제거할 수 있는 코드양이 늘어난다.
- 하지만 어떤 람다에서 매개변수 이름 자체가(count, incr) 개발자에게 좋은 가이드가 되기도 한다.
> - 더 읽기 쉽고 유지보수도 쉬워질 수 있다는 뜻이다. 

<br>

## 메소드 참조의 유형

#### 정적
- 정적 메소드를 가리키는 메소드 참조
- 메소드 참조 EX) `Integer::parseInt`
- 같은 기능의 람다 EX) `str -> Integer.parseInt(str)`

#### 한정적 인스턴스
- 수신 객체(참조 대상 인스턴스) 를 특정하는 한정적 인스턴스 메소드 참조
- 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑같다.
- 메소드 참조 EX) `Instant.now()::isAfter`
- 같은 기능의 람다 EX)

```java
Instant then = Instant.now();
t -> then.isAfter(t)
```

#### 비한정적 인스턴스
- 수신 객체(참조 대상 인스턴스) 를 특정하지 않는 비한정적 인스턴스 메소드 참조
- 함수 객체를 적용하는 시점에 수신객체를 알려준다.
- 수신 객체 전달용 매개변수가 매개변수 목록의 첫 번째로 추가되며, 그 뒤로는 참조되는 메소드 선언에 정의된 매개변수들이 뒤따른다. 
- 주로 스트림 파이프라인의 매핑, 필터 함수에서 사용된다. 
- 메소드 참조 EX) `String::toLowerCase`
- 같은 기능의 람다 EX) `str -> str.toLowerCase()`
 
#### 클래스 생성자
- 생성자 참조는 팩터리 객체로 사용된다.
- 메소드 참조 EX) `TreeMap<K, V>::new`
- 같은 기능의 람다 EX) `() -> new TreeMap<K, V>()`

#### 배열 생성자
- 생성자 참조는 팩터리 객체로 사용된다.
- 메소드 참조 EX) `int[]::new`
- 같은 기능의 람다 EX) `len -> new int[len]`

<br>

## 정리
- 메소드 참조하는 것이 명료하고 명확하다면 메소드 참조를 사용하고, 그렇지 않으면 람다를 사용하라.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
