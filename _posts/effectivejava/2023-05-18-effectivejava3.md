---
layout: post
title: '[EFFECTIVE JAVA] ITEM 3, private 생성자나 열거 타입으로 싱글턴임을 보증하라'
subtitle: ''
date: 2023-05-18 16:00:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 3, private 생성자나 열거 타입으로 싱글턴임을 보증하라

<br>

## 싱글턴(Singleton)
- 인스턴스를 오직 하나만 생성할 수 있는 클래스
- 무상태(stateless) 객체나 설계상 유일해야 하는 시스템 컴포넌트
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트가 테스트하기가 어려워질 수 있다.
> - 인터페이스를 구현해서 만든 싱글턴이 아니면 mock 구현으로 대체할 수 없기 때문이다.


## 싱글턴 만드는 방식
### 방식 1. public static 멤버가 final 필드인 방식

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }

    public void leaveTheBuilding() {...}
}
```

- private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출된다.
- 예외 : 권한이 있는 클라이언트가 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출
> - 방어법 : 생성자를 수정하여 두 번쨰 객체가 생성되려 할 때 예외를 던지게 한다. 

#### 장점
- 해당 클래스가 싱글턴임이 명백하게 들어난다.
- 간결하다.

<br>

### 방식 2. 정적 팩터리 메서드를 public static 멤버로 제공하는 방식

```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {...}
}
```

- Elvis.getInstance는 항상 같은 객체의 참조를 반환한다. (위의 리플렉션을 통한 예외는 똑같이 적용된다.)

#### 장점
- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점이다. 
- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다. (아이템30)

```java
public class GenericFactoryMethod {
public static final Set EMPTY_SET = new HashSet();

    public static final <T> Set<T> emptySet() {
        return (Set<T>) EMPTY_SET;
    }
}
```

> - 제네릭 싱글턴 팩토리 : 제네릭으로 타입 설정 가능한 인스턴스를 만들어두고, 반환 시에 제네릭으로 받은 타입을 이용해 타입을 결정

- 정적 팩터리의 메소드 참조를 공급자(supplier)로 사용할 수 있다는 점이다. ex) `Elvis::getInstance`

<br>

### 방식 3. Eunm 열거 타입 방식의 싱글턴

```java
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() {...}
}
```

#### 장점
- 간결하다.
- 추가 노력없이 직렬화 가능하다.
- 직렬화, 리플렉션 공격에도 제2의 인스턴스가 생기는 것을 막아준다.
- 대부분 원소가 하나 뿐인 열거 타입을 만드는 방법이 좋은 방법이다.

##### 조건
- 싱글턴이 Enum 클래스 외 다른 클래스를 상속해야 하면 사용할 수 없다. 

<br>

### 싱글턴 클래스의 직렬화
- 단순히 Serializable을 구현한다고 선언하는 것으로는 부족하다.
- 모든 인스턴스를 일시적(transient)이라고 선언하고 readResolve 메소드를 제공해야 한다. 
> - 이렇게 하지 않으면 직렬화된 인스턴스(객체 > 바이트 스트림)를 역직렬화(바이트스트림 > 객체)할 때마다 새로운 인스턴스가 생성된다.

```java
private Object readResolve() {
    return this.INSTANCE;
}
```

- 진짜 Elvis를 반환, 가짜 Elvis 를 가비지 컬렉터에 맡긴다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 깃헙](https://github.com/WegraLee/effective-java-3e-source-code)
- [쟈미의 이펙티브 자바 아이템3](https://jyami.tistory.com/58)
- [제네릭 싱글턴 팩토리_제이크서 위키 블로그](https://jake-seo-dev.tistory.com/13)