---
layout: post
title: '[EFFECTIVE JAVA] ITEM 33, 타입 안전 이종 컨테이너를 고려하라'
subtitle: ''
date: 2023-08-09 11:00:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 33, 타입 안전 이종 컨테이너를 고려하라

<br>

- 단일원소 컨테이너에서 매개변수화되는 대상은 (원소가 아닌) 컨테이너 자신이다. 따라서 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가 제한된다.
- 이보다 유연한 수단이 `타입 안전 이종 컨테이너 패턴` 이다.

<br>

## 타입 안전 이종 컨테이너 패턴 (type safe heterogeneous container pattern)
- 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 제공한다.
- 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해줄 것이다. 

```java
public class Favorites{ // 추상화
  public <T> void putFavorite(Class<T> type, T instance);
  public <T> T getFavorite(Class<T> type)
}
```

- 각 타입의 Class 객체를 매개변수화한 키 역할로 사용하면 된다.
- Class의 클래스가 제네릭이다.
> - class의 리터럴 타입은 Class<T> 다.
> - EX) String.class의 타입 Class<String>, Integer.class의 타입 Class<Integer>

#### 타입 토큰 
- 컴파일 타임 타입 정보와 런타임 타입 정보를 알아내기 위해 주고 메서드들이 주고 받는 class 리터럴 

<br>

```java
public class Favorites { // 구현
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

- 비한정적 와일드카드 타입이라 아무것도 넣을 수 없겠다고 생각할 수 있겠지만, 키가 와일드카드 타입이기 때문에 넣을 수 있다.
> - 모든 키가 서로 다른 매개변수화 타입일 수 있다 
> - EX) 첫 번째는 Class<String>, 두 번째는 Class<Integer> 일 수 있다. 

```java
public class Class<T>{
  T cast(Object obj);
}
```

- Map의 값들이 Object들이기 때문에 T로 변환이 필요하다. Class의 cast 메소드를 사용해 동적 형변환한다.
> - cast 메서드는 형변환 연산자의 동적 버전이다. 단순히 주어진 인수가 Class 객체가 알려주는 타입의 인스턴스인지 확인 후 맞으면 그대로 반환 아니면 ClassCastException을 던진다.
- cast 메소드의 시그니처가 Class 클래스가 제네릭이라는 이점을 완변하게 활용한다.
> - T 로 비검사 형변환하는 손실 없이도 Favorites 타입 안전하게 한다.  

<br>

## 타입 안전 이종 컨테이너 패턴의 제약

#### 악의적인 클라이언트가 Class 객체를 제네릭이 아닌 로 타입으로 넘기면 Favorites 인스턴스의 타입 안전성이 쉽게 깨진다.

```java
    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }
```

-  위처럼 put 시 동적 형변환을 넣어주면 된다.
- EX) java.util.Collections 에는 checkedSet checkedList, checkedMap 같은 메소드들
> - 제네릭과 로 타입을 섞어 사용하는 애플리케이션에서 클라이언트 코드가 컬렉션에 잘못된 타입의 원소를 넣지 못하게 추적하는 데 도움을 준다.

#### 실체화 불가 타입에는 사용할 수 없다.
- String, String[] 은 저장할 수 있어도 List<String>은 저장할 수 없다. List<String> 용 Class 객체를 얻을 수 없기 때문이다. 
> -  List<String> 과 List<Integer> 는 같은 Class 객체를 공유한다. 모두를 허용하여 객체 참조를 한다면 오류가 많아질 것이다.

<br>

## 한정적 타입 토큰
- 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한하는 타입 토큰
> - 타입 토큰 : 컴파일 타임 타입 정보와 런타임 타입 정보를 알아내기 위해 주고 메서드들이 주고 받는 class 리터럴 

#### 애너테이션 API는 한정적 타입 토큰을 적극적으로 사용한다.

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType) // AnnotatedElement 인터페이스에 선언된 메소드
```

> - 대상 요소에 달린 애너테이션을 런타임에 읽어오는 기능을 한다.
> - 이 메소드는 리플렉션의 대상이 되는 타입(클래스, 메소드, 필드) 같은 프로그램 요소를 표현하는 타입에서 구현한다. 

- annotationType 인수는 애너테이션 타입을 뜻하는 한정적 타입 토큰이다. 
- 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달려 있다면 그 애너테이션을 반환, 아니면 null 을 반환한다. 
- 즉, 애너테이션된 요소는 그 키가 애너테이션 타입인, 타입 안전 이종 컨테이너인 것이다. 

<br>

####  Class<?> 타입의 객체를 한정적 타입 토큰을 받는 메소드에 넘기고 싶을 때
- 객체를 Class<? extends Annotation> 으로 형변환할 수 있지만 컴파일하면 경고가 뜰 것이다.
- `asSubClass` 메소드 : 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다.

```java
static Annotation getAnnotation(AnnotationElement element, String annotationTypeName){
  Class<?> annotationType = null; //바한정적 타입 토큰
  try{
    annotationType = Class.forName(annotationTypeName);
  }catch (Exception ex){
    throw new IllegalArgumentException(ex);
  }
  return element.getAnnotation(annotationType.asSubClass(Annotation.class))
}
```

<br>

## 정리
- 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다.
- 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 유연한 타입 안전 이종 컨테이너를 만들 수 있다.
- 타입 안전 컨테이너는 Class 를 키로 사용하며, 이런 식으로 사용되는 Class 객체를 타입 토큰이라 한다. 또한 직접 구현한 키 타입도 사용 가능하다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
- [[Java] 제네릭과 와일드카드 타입에 대해 쉽고 완벽하게 이해하기(공변과 불공변, 상한 타입과 하한 타입) _ 망나니개발자](https://mangkyu.tistory.com/241)