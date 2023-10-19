---
layout: post
title: '[EFFECTIVE JAVA] ITEM 45, 스트림은 주의해서 사용하라'
subtitle: ''
date: 2023-10-19 11:40:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 45, 스트림은 주의해서 사용하라

<br>

## Stream API
- 다량의 데이터 처리 작업(순차적이든 병렬적이든)을 돕고자 JAVA 8 에 추가되었다.
- 데이터 원소의 유한, 무한 스퀀스를 의미한다.
- 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.
> - 스트림 안의 데이터 원소들은 객체 참조, 기본 타입 값(int, long, double)을 지원한다. 
- 메소드 연쇄를 지원하는 플루언트(fluent) API 이다.

<br>

### Stream pipeline
- 소스 스트림에서 시작해 종단 연산(terminal operation)으로 끝나며, 그 사이의 하나 이상의 중간 연산(intermediate operation)이 있을 수 있다.
> - 종단 연산 EX) : forEach(), collect(), match(), count(), reduce() 
> - 중간 연산 EX) : filter(), map(), sorted()
- 중간 연산은 스트림을 어떠한 방식으로든 변환한다. 
- 지연 평가(lazy evaluation) 된다. 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 

![sc](https://github.com/iheese/iheese.github.io/assets/88040158/8c9424fd-6f2a-485e-b377-3cce3b280d09)

- 실제로 필요하지 않은 데이터를 탐색하지 않는 것을 방지해 속도를 높인다. 이것을 Short-Circuit 이라 한다. 

<br>

## 스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다. 

```java
public class IterativeAnagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word),
                        (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

- computeIfAbsent 매소드 : 맵 안에 키가 있는지 찾은 다음, 있으면 단순히 그 키에 매핑된 값을 반환, 없으면 건네진 함수 객체를 키에 적용하여 값을 계산해낸 다음 그 키와 값을 매핑하고 계산된 값을 반환한다. 

```java
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```

- 맨 위 코드에서 스트림을 남발한 예시

```java
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

- 적절한 스트림 사용으로 가독성을 높였다. 

## 기존 코드는 스트림을 사용하도록 리팩토링하되, 새 코드가 나아 보일 때만 반영하자
- 스트림과 반복문을 적절히 조합하는 것이 최선이다. 

<br>

### char 값을 처리할 때는 스트림을 삼가는 편이 낫다

```java
"Hello world!".chars().forEach(System.out::print); // 72101108108111......
```

- 위를 출력하면 이상한 숫자가 나온다.
- chars() 가 반환하는 스트림 원소는 char 이 아니라 int 이기 때문이다. 

```java
"Hello world!".chars().forEach(x -> System.out.print((char) x)); // Hello world!
```

- 형변환을 해줘야 원하는 값을 얻을 수 있다. 

<br>

## 함수 객체로는 할 수 없지만 코드 블록으로 할 수 있는 일
- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 
> - 람다는 final 이거나 사실상 final인 변수만 읽을 수 있다. 지역변수 수정은 불가능하다.
- 코드 블록에서 return, break, continue 블록 반복의 흐름을 제어하거나 명시된 검사 예외를 던질 수 있다. 하지만 람다로는 불가능하다.

<br>

## 스트림에 안성맞춤인 일
- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링한다. 
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.
> - 더하기, 연결하기, 최솟값, 최댓값 등
- 원소들의 시퀀스를 컬렉션에 모은다.
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다. 

<br>

## 스트림으로 처리하기 어려운 일
- 파이프라인의 여러 단계를 통과할 때 각 단계의 값들에 동시에 접근하기는 어려운 경우이다.
> - 일단 한 값을 다른 값에 매핑하고 원래의 값은 읽는 구조이기 때문이다.

<br>

## 정리
- 스트림과 반복 방식, 결국은 개인 취향과 프로그래밍 환경의 문제이다.
- 스트림과 반복 방식, 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
- [Short-Circuit img](https://www.logicbig.com/tutorials/core-java-tutorial/java-util-stream/short-circuiting.html)
