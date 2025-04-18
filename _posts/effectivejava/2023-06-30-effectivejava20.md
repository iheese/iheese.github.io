---
layout: post
title: '[EFFECTIVE JAVA] ITEM 20, 추상 클래스보다는 인터페이스를 우선하라'
subtitle: ''
date: 2023-06-30 14:30:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 20, 추상 클래스보다는 인터페이스를 우선하라

<br>

- 자바8부터 인터페이스도 디폴트 메소드(default method) 제공 가능
- 추상 클래스 : 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다.
> - 단일 상속, 하위 클래스(상하 관계)
- 인터페이스 : 인터페이스가 선언한 메소드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 클래스를 상속했든 같은 타입으로 취급된다. 
> - 다중 상속, 같은 타입 취급

<br>

## 인터페이스의 장점

#### 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.
- 인터페이스 : 요구하는 메소드가 없다면 추가하고, 클래스 선언에 implements 구문만 추가하면 끝이다. 
- 추상 클래스 : 계층 구조상 조상이 되어, 클래스 계층 구조를 고려해야 한다. 

<br>

#### 믹스인 정의에 안성맞춤이다. 
- 믹스인(mixin) : 클래스가 구현할 수 있는 타입, 대상 타입의 주된 기능에 선택적 기능을 혼합한다는 의미
- 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.
> - EX) Comparable : 자신을 구현한 클래스의 인스턴스끼리는 순서를 정할 수 있다고 선언

- 추상 클래스 : 기존 클래스에 덧씌울 수 없고 클래스는 두 부모는 섬길 수 없다. 

<br>

#### 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

```java
public interface Singer {
    AudioClip sing(Song s);
}

public interface SongWriter {
    Song compose(int chartPosition);
}

public interface SingerSongWriter extends Singer, SongWriter {
    void strum();
    void actSensitive();
}
```

```java
public abstract class Singer {
    abstract void sing(String s);
}

public abstract class SongWriter {
    abstract void compose(int chartPosition);
}

public abstract class SingerSongWriter {
    abstract void sing(String s);
    abstract void Compose(int chartPosition);
    abstract void strum();
    abstract void actSensitive();
}
```

- 두 개의 인터페이스를 합치고 새로운 메소드까지 추가한 제 3의 인터페이스를 만들 수 있다. 
- 추상 클래스로 만들면 다중상속이 불가능하므로 새로운 추상 클래스를 만들어서 계층구조를 표현할 수 밖에 없다.
- 조합 폭발(combinatorial explosion) : 계층 구조를 만들기 위해 많은 조합이 필요하고 고도비만 계층구조가 생긴다.

<br>

#### 래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.
- 타입을 추상클래스로 정의하면 기능 추가 방법은 상속 뿐이다. 이는 활용도가 떨어지고 쉽게 깨진다.
- ITEM18, 상속보다는 컴포지션을 사용해라 의 래퍼 클래스 [참고](https://iheese.github.io/effectivejava/2023/06/28/effectivejava18/)

<br>

## 인터페이스의 디폴트 메서드 제약
- 디폴트 메소드를 제공할 때는 @implSpec 자바독 태그를 붙여 문서화해야 한다.
- equals, hashCode는 디폴트 메소드로 제공해선 안된다.
- 인터페이스는 인스턴스 필드를 가질 수 없다.
- public이 아닌 정적 멤버도 가질 수 없다.(private static 메소드 예외)
- 우리가 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다. 

<br>

## 인터페이스와 추상 골격 구현 클래스
- 인터페이스와 추상 골격 구현 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취한다. 
- 인터페이스 : 타입 정의 + 디폴트 메소드(필요하면)
- 골격 구현 클래스 : 나머지 메소드까지 구현
- 인터페이스 구현에 필요한 일이 대부분 완료 : 템플릿 메소드 패턴
- 골격 구현 클래스의 네이밍 관례 : Abstract+interface이름
> - EX) AbstractCollection, AbstractSet, AbstractList, AbstractMap

#### 장점
- 추상 클래스처럼 구현을 도와주면서, 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유롭다. 

<br>

## 시뮬레이트한 다중 상속(simulated multiple inheritance)
- 골격 구현 클래스를 우회적으로 사용한다.
- 다중 상속의 많은 장점을 제공하는 동시에 단점을 피하게 해준다. 
- 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메소드 호출을 내부 클래스의 인스턴스에 전달한다. 

<br>

## 골격 구현 작성 방법
1 . 인터페이스를 잘 살펴 다른 메소드들의 구현에 사용되는 기반 메소드를 선정한다.
2 . 기반 메서드들은 골격 구현에서는 추상 메소드가 되고, 기반 메소드들을 사용해 구현할 수 있는 메소드를 모두 디폴트 메소드로 제공한다.
> - equals, hashCode는 디폴트 메소드로 제공해선 안된다.
3 . 기반 메소드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어서 작성한다.
> - 인터페이스의 메소드 모두가 기반 메소드와 디폴트 메소드가 된다면 골격 구현 클래스로 별도로 만들 필요는 없다.
4 . 골격 구현은 기본적으로 상속해서 사용하는 것을 가정하므로 설계 및 문서화 지침을 모두 따라야 한다. 

```java
// 골격 구현 클래스 
public abstract class AbstractMapEntry<K,V>
        implements Map.Entry<K,V> {
    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
    @Override public V setValue(V value) {
        throw new UnsupportedOperationException();
    }
    
    // Map.Entry.equals의 일반 규약을 구현한다.
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(),   getKey())
                && Objects.equals(e.getValue(), getValue());
    }

    // Map.Entry.hashCode의 일반 규약을 구현한다.
    @Override public int hashCode() {
        return Objects.hashCode(getKey())
                ^ Objects.hashCode(getValue());
    }

    @Override public String toString() {
        return getKey() + "=" + getValue();
    }
}
```

<br>

## 단순 구현
- 골격 구현의 작은 변종이다.
- 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만 추상 클래스가 아니다.
- EX) AbstractMap.SimpleEntry

<br>

## 핵심 정리
- 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다.
- 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 고려하면 좋다.
- 골격 구현은 가능한 한 인터페이스의 디폴트 메소드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다.
> - 인터페이스 구현 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하다.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
