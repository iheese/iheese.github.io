---
layout: post
title: '[EFFECTIVE JAVA] ITEM 37, ordinal 인덱싱 대신 EnumMap을 사용하라'
subtitle: ''
date: 2023-08-22 11:40:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 37, ordinal 인덱싱 대신 EnumMap을 사용하라

<br>

## ordinal()울 배열 인덱스로 사용하는 것의 문제점

```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override public String toString() {
        return name;
    }
}
```

```java
Plant[] garden = {
            new Plant("바질",    LifeCycle.ANNUAL),
            new Plant("캐러웨이", LifeCycle.BIENNIAL),
            new Plant("딜",      LifeCycle.ANNUAL),
            new Plant("라벤더",   LifeCycle.PERENNIAL),
            new Plant("파슬리",   LifeCycle.BIENNIAL),
            new Plant("로즈마리", LifeCycle.PERENNIAL)
        };
```

```java
Set<Plant>[] plantsByLifeCycleArr =
        (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycleArr.length; i++)
    plantsByLifeCycleArr[i] = new HashSet<>();
for (Plant p : garden)
    plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p); // ordinal()을 배열 인덱스로 사용
// 결과 출력
for (int i = 0; i < plantsByLifeCycleArr.length; i++) {
    System.out.printf("%s: %s%n",
            Plant.LifeCycle.values()[i], plantsByLifeCycleArr[i]);
}
```

- 배열은 제네릭과 호환되지 않으니 비검사 형벼환을 수행해야 한다.
> - 컴파일이 깔끔하지 않다.
- 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아 표시해줘야 한다. 
- 정확한 정숫값을 사용한다는 것을 보증해야 한다.
> - 잘못된 값을 사용하면 잘못된 동작을 묵묵히 수행하거나 운이 좋으면 ArrayIndexOutOfBoundsException이 발생한다. 

<br>

## 올바른 해결책 : EnumMap
- EnumMap 은 열거타입을 키로 사용한다.

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
        new EnumMap<>(Plant.LifeCycle.class); //제네릭 타입 정보
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden)
    plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
```
- EnumMap 내부에서 배열을 사용하기 때문에 성능이 ordinal()을 사용했을 때와 비등하다.
> - 더 짧고 명료하고 안전하고 원래 버전과 성능도 비등하다.
- 안전하지 않은 형변환은 쓰지 않는다.
- 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공해주니 출력 결과에 직접 레이블을 달 일도 없다.
- 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 없다.

- EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로 런타임 제네릭 타입 정보를 제공한다. 

<br>

## EnumMap과 Stream을 함께 사용할 때

```java
Plant[] garden = {
            new Plant("바질",    LifeCycle.ANNUAL),
            new Plant("캐러웨이", LifeCycle.BIENNIAL),
            new Plant("딜",      LifeCycle.ANNUAL),
            new Plant("라벤더",   LifeCycle.PERENNIAL),
            new Plant("파슬리",   LifeCycle.BIENNIAL),
            new Plant("로즈마리", LifeCycle.PERENNIAL)
        };
```

```java
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle)));
```

- 위는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 공간과 성능 이점이 사라진다. 

```java
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle,
                () -> new EnumMap<>(LifeCycle.class), toSet())));
```

- 매개변수 3개짜리 Collector.groupingBy 메소드는 mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다. 
- 기존 EnumMap에서는 언제나 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 맵에 속한 값이 없다면 맵을 만들지 않는다.
> - EnumMap 버전은 해당하는 값이 없어도 ANNUAL, PERENNIAL, BIENNIAL 모두 만든다.
> - Stream는 해당하는 값이 없으면 해당 키의 맵을 만들지 않는다. 즉 ANNUAL, BIENNIAL 만 생성한다.

<br >

## 중첩 EnumMap 사용 예시

```java
public enum Phase {
    SOLID, LIQUID, GAS;
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
        // 새로운 상태를 추가하고 싶다면 그냥 아래처럼 추가하면 된다. 
//      PLASMA를 Phase 상태에 추가
//      IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);

        private final Phase from;
        private final Phase to;
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화한다.
        // 이전 상태에서 '이후상태에서 전이로의 맵'에 대응시키는 맵
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
        // 첫 번째 수집기인 groupingBy에서 전이를 이전 상태 기준으로 묶고
        // 두 번째 수집기인 toMap에서 이후 상태를 전이에 대응시키는 EnumMap 생성
        // (x, y) -> y는 선언만 하고 사용되진 않음, 단지 EnumMap을 얻기 위한 맵 팩터리

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
