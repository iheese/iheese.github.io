---
layout: post
title: '[EFFECTIVE JAVA] ITEM 6, 불필요한 객체 생성을 피하라'
subtitle: ''
date: 2023-05-23 21:00:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 6, 불필요한 객체 생성을 피하라

<br>

## 객체의 재사용
- 똑같은 기능의 객체를 매번 생성하기 보다는 객체 하나를 재사용하는 편이 나을 때가 많다.
- 재사용은 빠르고 세련되다.
- 불변 객체는 언제든지 재사용할 수 있다.

```java
String s = new String("java"); //해당 코드는 실행 될 때마다 String 인스턴스를 새로 만듬, Heap 영역에 존재
String s ="java"; //해당 코드는 하나의 String 인스턴스를 사용, String constant pool에서 검색해 재사용
```

<br>

## 불변 클래스의 정적 팩터리 메서드

```java
Boolean(String); //deprecated, 권장되지 않는 코드
Boolean.valueOf(String) //권장되는 코드
```

- 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 그렇지 않다.
- 불변 객체가 아니라 가변 객체라 해도 사용 중에 변경되지 않음을 알면 재사용할 수 있다.

<br>

## 생성 비용이 비싸다면 캐싱하여 재사용

```java
// 코드 6-1 성능을 훨씬 더 끌어올릴 수 있다!
static boolean isRomanNumeralSlow(String s) {
  return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                   + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

- 코드 6-1의 메소드 내부에서 만드는 정규표현식용 Pattern 인스턴스는 한 번 쓰고 버려져서 GC 대상이 된다. 
> - Pattern은 입력받은 정규표현식에 해당하는 유한 상태 머신(finite state machine)을 만들기 떄문에 인스턴스 생성 비용이 높다.
> - 유한 상태 머신(finite state machine) : 유한한 개수의 상태를 가질 수 있는 추상 기계 

```java
// 코드 6-2 값비싼 객체를 재사용해 성능을 개선한다.
private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
```

- Pattern 인스턴스를 클래스 초기화(정적 초기화) 과정에서 직접 생성하여 캐싱한다.
- 그 후 메서드가 호출될 때마다 해당 인스턴스를 재사용한다. 

- 해당 메서드가 호출되지 않았을 경우 지연 초기화(Lazy Initialization) 를 통해 불필요한 초기화를 없앨 수 있으나 권하지 않는다. 
- 코드를 복잡하게 하고 성능이 크게 개선되지 않는 경우가 많다.  

<br>

## 어댑터(뷰) 패턴 사용
- 실제 작업은 뒷단 객체에 위임하고 자신은 제2의 인터페이스 역할을 해주는 객체
> - 뒷단 객체 하나당 어댑터 하나씩

```java
Map<String, Object> map = new HashMap<>();
map.put("java", "effective")

//set1, set2 는 같은 인스턴스
Set<String> set1 = map.keySet();
Set<String> set2 = map.keySet();
```

- keySet()를 호출할 때마다 Set인스턴스가 만들어지지 않는다. 
- 뷰 객체를 여러 개 만들 것이라 생각할 수 있지만 매번 같은 인스턴스를 반환한다. 

<br>

## 오토박싱(Auto Boxing) 
- 오토박싱 : 기본 타입과 박싱된 기본 타입의 구분이 흐려지지만 완전히 없애진 못한다.
- 성능에 영향을 미친다. 

```java
private static long sum() {
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i; //  불필요한 Long 객체 생성
        return sum;
    }
```

- 박싱된 기본 타입보다는 기본 타입을 사용하고 의도치 않은 오토 박싱이 숨어들지 않게 조심해야 한다.

<br>

## 본인만의 객체 풀을 만들지 말자
- 아주 무거운 객체가 아닌데 단순히 객체 생성을 피하고자 객체 풀을 만들진 말자
> - 객체 풀 만들는 게 나은 예: DB 연결 
- JVM의 가비지 컬렉터는 상당히 최적화되어 있어서 직접 만든 객체 풀보다 빠르다. 

- 아이템50 : 기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라 : 방어적 복사(Defensive Copy) >  버그와 보안 구멍
- 아이템6 : 뷸필요한 객체 생성은 피하라 > 코드 형태와 성능에 영향

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 깃헙](https://github.com/WegraLee/effective-java-3e-source-code)