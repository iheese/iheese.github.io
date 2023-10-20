---
layout: post
title: '[EFFECTIVE JAVA] ITEM 46, 스트림에서는 부작용 없는 함수를 사용하라'
subtitle: ''
date: 2023-10-20 13:40:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 46, 스트림에서는 부작용 없는 함수를 사용하라

<br>

## 스트림 패러다임
- 핵심은 계산을 일련의 변환으로 재구성하는 부분이다.
- 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다.
> - 순수 함수 : 오직 입력만이 결과에 영향을 주는 함수를 말한다. 
- 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다. 이렇게 하려면 (중간, 종단 단계든) 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야 한다. 

<br>

### forEach 예제

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

- 위는 텍스트 파일에서 단어별 수를 세어 빈도표를 만드는 일을 하는 스트림이다. 
- 위는 스트림 코드를 가장한 반복적인 코드이다.
> - 조금 더 길고, 가독성이 좋지 않고, 유지보수에도 좋지 않다. 
- 모든 작업이 종단 연산인 forEach에서 일어나는데 외부 상태(freq)을 변경하면서 문제가 생긴다. 

```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words
            .collect(groupingBy(String::toLowerCase, counting()));
}
```

- 위 코드를 스트림답게 올바르게 작성하였다.
> - 짧고 명확해졌다.

#### forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자
- forEach 종단 연산은 기능이 적고, 가장 덜 스트림답다. 
> - 대놓고 반복적이라 병렬화할 수도 없다. 
- 가끔 스트림 계산 결과를 기존 컬렉션에 추가하는 등의 코드로 사용할 수는 있다.

<br>

## collector 수집기 예제

### toList

```java 
//  Map<String, Long> freq; ...
List<String> topTen = freq.keySet().stream()
        .sorted(comparing(freq::get).reversed())
        .limit(10)
        .collect(toList());
```

- freq 변수명의 Map에서 value값을 추출해서 역순으로 정렬한다.
- 10개를 뽑아 리스트에 담는다. 
- toList() 원소를 리스트에 담아 만들어준다. 

<br>

### toMap

#### 인수 2개를 받는 toMap
- toMap(keyMapper, valueMapper)

```java
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(
        toMap(Object::toString, e -> e)
    );
```

- 열거 타입 상수의 문자열 표현을 열거 타입 자체에 매핑하여 Map으로 리턴한다. 
- 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다.
- 스트림 원소 다수가 같은 키를 사용한다면 파이프라인이 IllegalStateException을 던지며 종료한다. 

<br>

#### 인수 3개를 받는 toMap
- 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다.

```java
Map<Artist, Album> topHits = albums.collect(
    toMap(Album::artist, a -> a, maxBy(comparing(Album::sales)))
);
```

- maxBy 는 Comparator<T> 를 입력받아 BinaryOperator<T>를 돌려준다. 
- 앨범 스트림을 맵으로 바꾸는데, 각 음악가와 그 음악가의 베스트 앨범(판매량 기준)으로 짝지는 것이다. 

<hr>

- 충돌이 나면 마지막 값을 취하는 (last-write-wins) 수집기

```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

```java
// Stream 학습 내용 중 예제
public Map<String, Integer> quiz1() throws IOException {
    List<String[]> csvLines = readCsvLines();

    return csvLines.stream()
            .map(line -> line[1].replaceAll("\\s", ""))
            .flatMap(hobbies -> Arrays.stream(hobbies.split(":")))
            .collect(Collectors.toMap(hobby -> hobby, hobby -> 1, (oldValue, newValue) -> newValue += oldValue));
    }
```

- 많은 스트림의 결과가 비결정적이므로 매핑 함수가 키 하나에 연결해준 값이 모두 같을 때, 값이 다르더라도 모두 허용되는 값일 때 동작하는 수집기가 필요하다.

<hr>

- 그 외 toCocurrentMap은 병렬 실행 된 후 ConcurrentHashMap 인스턴스를 생성한다.

<br>

### groupingBy
- 입력으로 분류 함수(classifier)을 받고 출력으로 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다.

#### 분류 함수 1개를 받는 groupingBy

```java
// 결과 Map<String, List<String>>, Key가 alphabetize된 결과 
Map<String, List<String>> collect1 = 
    words.collect(groupingBy(word -> alphabetize(word)))
```

- alphabetize 결과가 같은 단어들의 리스트로 매핑하는 맵을 생성한다. 
> - item45의 아나그램 프로그램에서 사용된 수집기

<br>

#### 분류함수와 다운스트림 수집기를 받는 groupingBy
- 다운스트림의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 일이다. 

```java
// 결과 Map<String, List<String>>, Key가 alphabetize된 결과 
Map<String, Set<String>> collect2 = 
  words.collect(groupingBy(HybridAnagrams::alphabetize, toSet()));
```

- Map의 값을 List 가 아닌 Set으로 하는 코드
- toSet() 이 아닌 toCollection(collectionFactory) 를 건네는 방법으로 유연하게 컬렉션 타입을 선택할 수 있다.

<hr>

```java
Map<String, Long> freq = collect3
    .collect(groupingBy(HybridAnagrams::alphabetize, counting()));
```

- 키에 대한 카테고리에 속하는 원소의 갯수를 매핑한 맵을 얻을 수 있다.

<br>

#### 분류함수와 다운스트림, 맵 팩터리를 받는 groupingBy
- 이 메소드는 점층적 인수 목록 패턴(telescoping argument list pattern) 에 어긋난다. 
- 죽 macFactory 매개변수가 downStream 매개변수보다 앞에 놓인다.

```java
TreeMap<String, Long> freq = words
    .collect(groupingBy(HybridAnagrams::alphabetize, TreeMap::new, counting()));
```

- 인수가 3개인 groupingBy 를 사용하면 맵과 그 안에 담긴 컬렉션 타입을 모두 지정할 수 있다.

<hr>

- 총 세 가지 groupBy에 대응하는 groupingByConcurrent 메소드가 있으며, 동시 수행 버전으로 ConcurrentHashMap을 리턴한다.
- 잘 쓰이진 않지만 partitioningBy도 있다. 분류 함수 자리에 predicate를 받고 키가 Boolean인 맵을 반환한다. 
> - 다운스트림 수집기까지 입력받는 버전도 다중정의되어 있다. 

#### Stream의 count 메소드를 직접 사용하면 될테니 collect(counting()) 을 사용할 일은 없다. 
- counting() 같이 스트림의 기능을 일부 복제하여 다운스트림 수집기를 작은 스트림으로 동작하게 한 메소드들이 많다.

<br>

### Collectors에 정의되어 있지만 수집과는 관계가 없는 메소드 

### minBy, maxBy

```java
Map<Artist, Album> topHits = albums.collect(
    toMap(Album::artist, a -> a, maxBy(comparing(Album::sales)))
);
```

- 인수로 받은 비교자들 통해 가장 크거나 작은 원소를 반환한다. 

<br>

### joining
- CharSequence 인스턴스의 스트림에만 적용할 수 있다.

```java
public class Joining {
    public static void main(String[] args) {
        List<String> stringList = List.of("apple", "banana", "pear");
        // 인수가 없는 경우
        String collect1 = stringList.stream().collect(joining());
        System.out.println(collect1); // applebananapear

		// 인수가 1개인 경우
        String collect2 = stringList.stream().collect(joining(","));
        System.out.println(collect2); // apple,banana,pear

		// 인수가 3개인 경우
        String collect3 = stringList.stream().collect(joining(",","[","]"));
        System.out.println(collect3); // [apple,banana,pear]
    }
}
```

- 매개변수가 없을 때 : 단순히 원소 연결
- 매개변수 1개 일 때 : 매개변수가 구분문자가 되어 문자열 생성
- 매개변수 3개 일 때 : 구분 문자, 접두문자(prefix), 접미문자(suffix) 를 받는다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
- [Short-Circuit img](https://www.logicbig.com/tutorials/core-java-tutorial/java-util-stream/short-circuiting.html)
