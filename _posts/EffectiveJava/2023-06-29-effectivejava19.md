---
layout: post
title: '[EFFECTIVE JAVA] ITEM 19, 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지해라'
subtitle: ''
date: 2023-06-29 15:30:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 19, 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지해라

<br>

## 상속을 고려한 설계와 문서화

#### 상속용 클래스는 재정의할 수 있는 메소드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.
- 호출 메소드에 API 설명에 기술, 호출 순서, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야 한다.
- 재정의 가능 메소드를 호출할 수 있는 모든 상황을 문서로 남겨야 한다.
> - EX) 백그라운드 스레드나 정적 초기화 과정에서도 호출이 일어날 수 있다. 

- API 문서의 메소드 설명 끝에 Implementation Requirements : 메소드 내부 동작 방식 설명
> - 메소드 주석에 @implSpec 태그 붙여주면 자바독 도구가 생성해준다. 

#### "좋은 API 문서란 '어떻게' 가 아닌 '무엇'을 하는지를 설명해야 한다" 와 대치
- 상속이 캡슐화를 해치기 때문이다. 
- 쿨랴스를 안전하게 상속할 수 있도록 하려면 내부 구현 방식을 설명해야 한다. 

<br>

## 상속 설계시 내부 동작 과정에 훅 선별
- 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메소드 형태로 공개해야 할 수도 있다.
- EX) java.util.AbstractList의 removeRange 메소드는 하위 클래스의 부분 리스트의 clear 메소드를 고성능으로 만들기 위해 존재한다.
- 실제 하위 클래스를 만들어 시험해본 뒤, protected 메소드의 내부 노출 정도를 정해야하고 그 수가 가능한 적어야 한다.
- 한편으로 너무 적게 노출해서 상속으로 얻는 이점마저 없애지 않도록 주의해야 한다. 

#### 상속용 클래스를 시험하는 방법은 직접 하위 클래스로 만들어보는 것이 유일하다.
- 꼭 필요한 protected 멤버를 놓치면 하위 클래스 작성할 때 빈 자리가 드러나다.
- 하위 클래스를 여러 개 만들 때까지 쓰이지 않는 protected 멤버는 private으로 돌린다.

#### 배포 전 하위 클래스를 작성하여 검증해보자!

<br>

## 상속용 클래스의 생성자는 재정의 가능 메소드 호출 금지
- 상속용 클래스의 생상자는 직접적으로든 간접적으로든 재정의 가능 메소드를 호출해서는 안된다. 
1. 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행된다.
2. 하위 클래스에서 재정의한 메소드가 하위 클래스의 생성자보다 먼저 호출된다.
3. 그 재정의한 메소드가 하위 클래스의 생성자에서 초기화하는 값에 의존한다면 문제가 생길 것이다. 

```java
public class Super {
    // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
    }
}
```

```java
// 생성자에서 호출하는 메서드를 재정의했을 때의 문제
public final class Sub extends Super {
    // 초기화되지 않은 final 필드. 생성자에서 초기화
    private final Instant instant;

    Sub() {
        instant = Instant.now();
    }

    // 재정의 가능 메서드. 상위 클래스의 생성자가 호출
    @Override public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe(); // null, instant
    }
}
```

- private, final, static 메소드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다. 

<br>

## Cloneable, Serializable 인터페이스는 상속 설계에 방해가 된다.
- 들 증 하나라도 구현한 클래스를 상속할 수 있게 설계하는 것은 일반적으로 좋은 생각이 아니다.

- clone, readObject 메소드는 생성자와 비슷한 효과를 낸다.(새로운 객체를 만듬)
- 그래서 이들을 구현할 때 따르는 제약도 생성자와 비슷하다는 것에 주의해야 한다. 

#### clone, readObject 모두 직간접적으로 재정의 가능 메소드를 호출해선 안된다.
- readObject : 하위 클래스의 상태가 미처 역직렬화되기 전에 재정의한 메소드부터 호출한다.
- clone : 하위 클래스의 clone 메소드가 복제본의 산태를 올바르게 수정하기 전에 재정의 메소드를 호출한다.  
> - clone이 완벽하지 못했어서 복제본 내부 어딘가에서 여전히 원본 객체의 데이터를 참조하고 있다면 원본 객체에도 피해를 줄 수 있다.

<br>

## Serializable을 구현한 상속용 클래스 주의 
- readResolve나 writeReplace 메서드를 갖는다면 이 메소드들은 private이 아닌 protected로 선언해야 한다. 
> - priavte으로 선언하면 하위 클래스에서 무시되기 때문이다. 이 역시 상속을 허용하기 위해 내부 구현을 클래스 API로 공개하는 예 중 하나이다. 

<br>

## 일반적인 구체 클래스의 상속 설계와 조언

#### 상속용으로 설계되지 않은 클래스는 상속을 금지한다.
- 클래스를 final로 선언한다.
- 모든 생성자를 private, package-private으로 선언하고 public 정적 팩터리를 만들어준다.
> - 내부에서 다양한 하위 클래스를 만들어 쓸 수 있는 유연성을 준다.

#### 구체 클래스 상속보다는 인터페이스 상속을 사용하는 것이 더 나은 대안이다.
- EX) Set, List, Map

#### 상속을 허용해야겠다면 재정의 가능 메소드를 사용하지 않게 하고 이 사실을 문서로 남긴다.
- 구체 클래스가 표준 인터페이스를 구현하지 않았는데 상속을 허용하고 싶다면 재정의 가능 메소드를 사용하지 않게 하고 문서화한다. 

#### 클래스의 동작을 유지하면서 재정의 가능 메소드를 사용하는 코드를 제거할 수 있는 방법
- 재정의 가능 메소드는 자신의 본문 코드를 private 도우미 메소드로 옮긴다.
- 이 도우미 메소드를 호출한다. 

```java
public method () {
    helpingMethod();
}

private helpingMethod () {
    ...
}
```

<br>

## 핵심 정리
- 상속용 클래스 내부에서 스스로 어떻게 사용하는지 모두 문서로 남겨야 하고 그 클래스가 쓰이는 동안 꼭 지켜야 한다.
- 다른 개발자들이 효율 좋은 하위 클래스를 만들 수 있도록 일부 메소드를 protected로 제공해야 할 수도 있다.
- 클래스를 확장할 명확한 이유가 없다면 상속을 금지해야 한다.
- 상속을 금지할 때는 클래스를 final로 선언하거나 생성자 모두를 외부에서 접근할 수 없도록 만들면 된다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
