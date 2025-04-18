---
layout: post
title: '[EFFECTIVE JAVA] ITEM 13, clone 재정의는 주의해서 진행하라'
subtitle: ''
date: 2023-06-12 12:30:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 13, clone 재정의는 주의해서 진행하라

<br>

## Cloneable 인터페이스

- Cloneable : 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스(mixin interface)
> - 믹스인(mixin) : 다른 클래스에서 사용할 목적으로 만들어진 클래스, 포함으로 설명되며 상속의 단점을 해결해준다. 
> > - 클래스가 구현할 수 있는 타입, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.(ITEM20)

## 문제
- clone 메소드가 선언된 곳이 Cloneable이 아닌 Object이고 protected임에 있다.
- Cloneable을 구현하는 것만으로는 clone 메소드를 호출할 수 없다.

<br>

## Cloneable 인터페이스의 역할 
- Object의 protected 메소드인 clone의 동작 방식을 결정한다. 
- Cloneable을 구현한 클래스의 인스턴스, clone 메소드 호출 > 객체의 필드들을 하나하나 복사한 객체 반환
- Cloneable을 구현하지 않은 상태에서 clone 메소드 호출 > CloneNotSupportException
- 상위 클래스에 정의된 protected 메소드의 동작 방식을 변경한 것이다.
> - 인터페이스를 이례적으롯 사용한 예, 따라 하지 말자.

<br>

## clone 메소드의 일반 규약
- `x.clone() != x` : 참
- `x.clone().getClass() == x.getClass()` : 참
- `x.clone().equals(x)` : 참, 필수는 아님
- `x.clone().getClass() == x.getClass()` : 관례상 super.clone을 호출해야하는데 그 관례를 따른다면 참

<br>

## clone 메소드의 사용

- 생성자 연쇄(constructor chaining) 와 비슷한 메커니즘, 상속으로 부모 클래스의 기본 생성자를 계속 호출 > clone도 상속으로 연쇄해서 객체를 생성해야 한다. 
- 하위 클래스에서 super.clone을 호출하면 잘못된 클래스의 객체가 만들어져 하위 클래스의 clone 메소드가 제대로 작동하지 않는다. 
- clone을 재정의한 클래스가 final이라면 걱정해야 할 하위 클래스는 없으니 이 관례는 무시해도 안전하다.

<br>

## 모든 필드가 기본 타입이거나, 불변 객체일 때, 제대로 상위 클래스를 상속하여 Cloneable 구현 
- super.clone 호출한다.
> - 원본 필드와 똑같은 값을 가진다.
- 쓸데없는 복사 지양을 위해 불변 클래스는 clone 메소드를 제공하지 않는 게 좋다.

```java
@Override public PhoneNumber clone(){
  try{
    return (PhoneNumber) super.clone();
  }catch(CloneNotSupportedException e){
    throw new AssertionError(); //PhoneNumber 클래스에 Cloneable만 구현되어 있으면 있어날 수 없는 일
  }
}
```

- PhoneNumber 클래스 선언에 Cloneable 구현 필수 
- 자바의 공변 반환 타이핑 덕분에 가능하고 이 방식을 권장한다.
> - 공변 변환 타이핑(covariant return typing) : 부모 클래스의 메소드를 오버라이딩하는 경우, 부모 클래스의 반환 타입은 자식 클래스의 타입으로 변경 가능하다. 
> > - T'가 T의 subType이면, C<T'>는 C<T>의 SubType이다.

<br>

## 가변 객체일 때, Cloneable 구현

```java
public class Stack{
  private Object[] elements;
  private int size;
}

@Override public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

- clone 메소드가 단순히 불변 객체를 복사할 때와 같이 super.clone의 결과를 그대로 반환한다면 size는 올바른 값을 가지겠지만, elements 필드는 원본 Stack 인스턴스와 같은 배열을 참조하게 된다.
- 위처럼 하지 않으면 복사한 객체는 elements 참조값을 가지고 있게 되어 원본객체를 수정하면 복사객체도 수정되는 현상이 일어날 수 있다. 
- clone 메소드는 사실상 생성자와 같은 효과를 낸다. 즉 clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다. 

### 배열의 복제
- 배열의 clone 메소드는 권장되고 있다.
- 배열은 clone 기능을 제대로 사용하는 유일한 예이다.

<br>

## HashTable용 cloneable 구현(복잡한 가변 객체)
- 해시테이블 내부는 버킷들의 배열, 각 버킷은 키-값 쌍을 담는 연결 리스트의 첫 번째 엔트리를 참조한다. 
- 위의 Stack처럼 복제하면 자기 자신만의 버킷 배열을 갖지만, 원본과 같은 연결 리스트를 참조하여 예기치 못하게 동작할 수 있다. 
- 그래서 각 버킷을 구성하는 연결 리스트를 복사해야 한다.  

![hashtable](https://github.com/iheese/iheese.github.io/assets/88040158/0b4ef536-1cfd-4b27-b27e-bbb21b725d58)

1 . 해시테이블의 clone 메소드는 적절한 크기의 새로운 버킷 배열을 할당한다.
2 . 다음 원래의 버킷 배열을 순회하며 비지 않은 버킷에 대해 깊은 복사(HashTable.Entry의 deep copy)를 수행
- HashTable.Entry의 deep copy : 자신이 갈리키는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출한다.
> - 버킷이 길기 않다면 잘 작동한다. 하지만 그렇게 좋은 방식은 아님 > 재귀호출 때문에 스택 프레임을 소비하여 리스트가 길면 스택 오버플로를 일으킬 수 있다.
- 그래서 deepCopy를 재귀 방식이 아닌 반복자를 써서 순회하는 방식으로 수정해야 한다. 
3 . super.clone을 호출하여 얻은 객체의 모드 필드를 초기 상태로 설정 >  원복 객체의 상태를 다시 생성하는 고수준 메소드들을 호춣한다. 
- EX) HashTable에서는  `put(key, value)`
> - 간단한 코드를 얻을 수 있지만 저수준 API보다는 느리고 필드 단위 객체 복사를 우회하므로 전체 Cloneable과 어울리지 않는 방식이기도 하다. 

<br>

## 상속을 위한 클래스에서 clone 메소드 주의 사항

- 상속용 클래스는 Cloneable을 구현하면 안된다. 
- Cloneable 구현 여부를 하위 클래스에서 선택하도록 해준다.
- clone을 동작하지 못하게 구현하고 하위 클래스에서 재정의하지 못하게 한다, 아래처럼 퇴화시켜 놓는다. 

```java
@Override
protected final Object clone() throws CloneNotSupportedException{
  throw new CloneNotSupportedException();
}
```

<br>

### 추가적인 주의 사항

- 생성자에서 재정의될 수 있는 메소드를 호출하지 말아야 하는데 (ITEM 19) clone도 재정의될 수 있는 메소드를 호출하면 안된다. 
> - clone이 하위 클래스에서 재정의한 메소드를 호출하면 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃는다. > 원본, 복제본 상태가 달라짐
- 재정의한 public clone 메소드에서는 throws 절을 없애야 사용하기 편하다. 
- Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메소드 역시 적절히 동기화해줘야 한다. 

<br>

## clone 메소드 재정의 방법
- Cloneable을 구현하는 클래스는 clone을 재정의해주어야 한다.
- 이 때 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경한다.
- 가장 먼저 super.clone을 호출한 후 필요한 필드를 적절히 수정한다.
- 객체 내부 깊은 구조에 숨어 있는 가변 객체를 복사하고 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 함을 뜻한다.
- 기본 타입 필드, 불변 객체 참조만 갖는 다면 아무 필드도 수정할 필요가 없다
- 일련 번호, 고유 ID는 기본타입, 불변이어도 수정해야 한다. 

<br>

## clone 메소드 재정의 다른 방법

- 복사 생성자 : 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.

```java
public Yum(Yum yum){...};
```

- 복사 팩터리 : 자신과 같은 클래스의 인스턴스를 인수로 받는 메소드를 제공하는 방법이다. 

```java
public static Yum newInstance(Yum yum){...};
```

### 위 방법의 장점
- 위험한 객체 생성 메커니즘(셍성자를 쓰지 않는 방식)을 사용하지 않는다.
- 정상적인 final 필드 용법과도 충돌하지 않는다.
- 엉성한 문서화 규약에 기대지 않는다.
- 불필요한 검사 예외도 없고 형변환도 필요하지 않다.
- 해당 클래스가 구현한 인터페이스 타입의 인트턴스를 인수로 받을 수 있다. 
> - 인터페이스 기반 복사 생성자 = 변환 생성자 (conversion constructor)
> - 인터페이스 기반 복사 팩터리 = 변환 팩터리 (conversion factory)

<br>

## 결론
- 새로운 인터페이스를 만들 때 Cloneable을 확장하지 말자
- 새로운 클래스를 만들 때도 이를 구현하지 말자
- 기본 원칙은 복제 기능은 생성자와 팩터리를 이용하되, 배열은 clone 메소드 방식이 깔끔하고 합당한 예외에 속한다.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
- [자바 믹스인(mixins)이란?_제이크서 위키 블로그](https://jake-seo-dev.tistory.com/30)