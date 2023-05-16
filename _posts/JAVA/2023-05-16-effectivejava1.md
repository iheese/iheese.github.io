---
layout: post
title: '[EFFECTIVE JAVA] ITEM 1, 생성자 대신 정적 팩터리 메소드를 고려하라'
subtitle: ''
date: 2023-05-16 12:00:00 +0900
categories: 'JAVA'
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 1, 생성자 대신 정적 팩터리 메소드를 고려하라

<br>

## 장점
### 1 . 이름을 가질 수 있다.
- 생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명할 수 없다. 정적 팩터리 메소드는 이름만 잘 지으면 반환될 객체의 특성을 잘 묘사할 수 있다.
> - `BingInteger(int, int, Random)` vs `BingInteger.probablePrime()`

- 생성자의 시그니처 (같은 매개변수 타입, 갯수)가 같은 것이 여러 개 반복될 것 같으면 정적 팩터리 메소드 사용을 고려해보자
> - `Person.walk(int x, int y)`
> - `Person.move(int x, int y)`

<br>

### 2 . 호출 될 때마다 인스턴스를 새로 생성하지 않아도 된다.
- static 자원은 JVM 클래스로더의 초기화 부분에서 할당된다.
> - `Boolean.valueOf(boolean)` : 해당 메소드는 객체를 생성하지 않는다. 
- 인스턴스를 통제하면 클래스를 싱글턴으로 만들 수도, 인스턴스화 불가로 만들 수 있다. 또한 불변 클래스에서 동치인 인스턴스가 하나임을 보장할 수 있게 된다. 
> - 플라이웨이트 패턴 : 동일하거나 유사한 객체들 사이에 가능한 많은 데이터를 서로 공유하여 메모리 사용량을 최소화하는 패턴

<br>

### 3 . 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
- 반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 보장하게 된다. 
- 구현 클래스를 공개하지 않고도 객체를 반환할 수 있어 API를 작게 유지할 수 있다. 

<br>

### 4 . 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
- 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관이 없다.
> - `EnumSet` : public 생성자 존재 x, 정적 팩터리만 제공, 원소 64개 이하면 RegularEnumSet 인스턴스를, 65개 이상이면 JumboEnumSet 인스턴스를 반환한다. 

<br>

### 5 . 정적 팩터리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
- 서비스 제공자 프레임워크(service provider framework)를 만드는 근간, 대표적으로 JDBC(Java DataBabe Connectivity)
- 반환값이 인터페이스여도 되고, 반환값의 구현체기만 하면 리턴 가능하다. > 유연한 시스템 구현 가능

```
// InterfaceReturnValue는 인터페이스여도 아래와 같이 메소드 작성 가능
public static List<InterfaceReturnValue> getReturnValue(){
    return new ArrayList<>();
}
```

<br>

#### service provider framework의 컴포넌트
- 서비스 인터페이스(Service Interface) : 구현체의 동작 정의, JDBC: Connection
- 제공자 등록 API(Provider Registration API) : 제공자가 구현체를 등록할 때 사용, JDBC: DriverManager.registerDriver
- 서비스 접근 API(Service Access API) : 클라이언트가 서비스의 인스턴스를 얻을 때 사용, JDBC: DriverManager.getConnection
> - 클라이언트는 원하는 구현체의 조건을 명시할 수 있다. 
- 서비스 제공자 인터페이스(Service Provider Interface): Service Interface의 인스턴스를 생성하는 팩터리 객체, JDBC : Driver

<br>

## 단점
### 1 . 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메소드만 제공하면 하위 클래스를 만들 수 없다. 
- 상속으로 일어날 수 있는 문제를 컴포넌트 사용을 유도하고 불변 타입으로 만들려면 제약을 지켜야 하므로 장점이라고 할 수 있다.

<br>

### 2 . 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
- 사용자는 정적 팩터리 메소드 방식 클래스를 인스턴스화 할 방법을 알아내야 한다. 
> - from: 매개변수를 받아서 해당 타입의 인스턴스 반환. 형변환.
> - of: 여러 매개변수를 받아 적합한 인스턴스 반환
> - valueOf: from, of 보다 자세한 버전
> - instance, getInstance: 매개변수 인스턴스를 반환하지만 보장하지는 않음
> - create, newInstance: 매번 새로운 인스턴스 생성해 반환
> - getType: 반환 타입과 팩터리메서드 클래스가 다름. Type은 반환 타입 명시
> > - `FileStore fs = Files.getFileStore(path)`
> - type: getType, newType 간결한 버전
> > - `Collections.list(legacyLitany)`

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)