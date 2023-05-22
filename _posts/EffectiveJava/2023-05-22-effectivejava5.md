---
layout: post
title: '[EFFECTIVE JAVA] ITEM 5, 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라'
subtitle: ''
date: 2023-05-22 15:20:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 5, 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

<br>

### 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다. 

```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    private SpellChecker() {} //객체 생성 방지
}
```

- 정적 유틸리티를 잘못 사용한 예, 유연하지 않고 테스트가 어려움

```java
//싱글턴
public class SpellChecker{
  private final Lexicon dictionary = ...;
  private SpellChecker(...){}
  public static SpellChecker INSTANCE = new SpellChecker(...);
}
```

- 싱글턴을 잘못 사용한 예
- 두 가지 예 모두 주입받는 dictionary가 하나로만 지정되어 여러 사전을 사용할 수 없다는 점은 큰 단점이다.
- 이를 해결하기 위해 의존 객체 주입 형태가 올바르다. 


```java
//의존 객체 주입 형태
public class SpellChecker{
  private final Lexicon dictionary;
  
  public SpellChecker(Lexicon dictionary){
    this.dictionary = Objects.requireNonNull(dictionary);
  }
}
```

- 의존 객체 주입 형태
- dictionary 필드의 final을 없애고 다른 사전으로 교체하는 메서드를 추가할 수 있으나, 이는 어색하고 오류를 내기 쉽고 멀티스레드 환경에서 사용할 수 없다. 
- 불변을 보장한다.
- 생성자, 정적 팩토리, 빌더에 똑같이 응용이 가능하다. 

<br>

### 생성자에 자원 팩터리를 넘겨주는 방식
- 팩터리 : 호출할 떄마다 특정 타입의 인스턴스를 반복헤서 만들어주는 객체
> - 팩터리 메소드 패턴(Factory Method Pattern)을 구현한 것이다.
- 자바8의 `Supplier<T>` 인터페이스가 예시
> - 위 인터페이스로 입력 받는 메서드는 일반적으로 한정적 와일드 카드(Bounded Wildcard Type)을 사용해 팩터리 타입 매개변수를 제한해야 한다.

```java
Mosaic create(Supplier<? extends Tile> tileFactory){...}
```

- Tile 클래스를 상속받은 하위 클래스를 넣어 Mosaic 클래스를 생성할 수 있다.

<br>

### 의존 객체 주입 프레임 워크
- 의존 객체 주입이 유연성과 테스트의 용이성을 개선해주지만 의존성이 수천 개나 되는 프로젝트는 코드를 어지럽게 만든다.
- 이는 대거(Dagger), 주스(Guice), 스프링(Spring) 같은 의존 객체 주입 프레임워크를 통해 의존 객체를 직접 주입하도록 설계된 API를 알맞게 응용하여 사용하고 있다.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 깃헙](https://github.com/WegraLee/effective-java-3e-source-code)