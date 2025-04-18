---
layout: post
title: '[EFFECTIVE JAVA] ITEM 18, 상속보다는 컴포지션을 사용하라'
subtitle: ''
date: 2023-06-28 16:00:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 18, 상속보다는 컴포지션을 사용하라

<br>

### 상속 안전할 때 
- 상위 클래스와 하위 클래스를 모두 같은 프로그래머가 통제하는 패키지 안에서 사용될 때
- 확장할 목적으로 설꼐되었고 문서화도 잘 되어 있을 때

### 상속 안전하지 않을 때
- 다른 패키지의 구체 클래스를 상속할 때

<br>

## 하위 클래스가 깨지기 쉬운 이유

### 상속은 캡슐화를 깨뜨린다. 
- 상위 클래스 구현에 따라 하위 클래스의 동작에 이상이 생길 수 있다.
- 상위 클래스는 릴리스마다 내부 구현에 변경이 생길 수 있고, 그 여파로 하위 클래스가 오작동할 수 있다.
- 이러한 이유로 상위 클래스 설계자가 확장을 충분히 고려하고 문서화도 제대로 해놓아야 한다. 

<br>

### 자신이 다른 부분을 사용하는 자기사용(self-use) 여부

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount()); //3 예상 // 6 리턴
    }
}
```

- 위의 getAddCount 메소드를 사용하면 제대로 작동하지 않는다. 
- HashSet의 addAll 가 add 메소드를 사용해 구현되어 있어 addCount 값이 중복되어 더해진다. 
- 이는 해당 클래스의 내부 구현 방식에 해당되며, 다음 릴리스에서도 유지될지 알 수 없다.  

<br>

### 상위 클래스의 메소드를 재구현
- addAll 메소드를 다른 방식으로 재정의한다.
> - 주어진 컬렉션을 순회하면서 add 메소드를 한번씩 호출
- 상위 클래스의 메소드를 재구현하는 것은 시간이 더 들고, 자칫 오류를 내거나 성능 이슈가 생길 수 있다.
- 하위 클래스에서는 접근할 수 없는 private 필드를 사용하면 아예 사용할 수 없다. 

<br>

### 다음 릴리즈에서 상위 클래스에 새로운 메소드 추가
- 상위 메소드에 또 다른 원소 추가 메소드가 생성된다면, 하위 클래스에서 재정의하지 못한 그 새로운 메소드를 이용해 허용되지 않은 원소를 추가할 수 있게 된다.

<br>

### 클래스를 확장하더라도 메소드 재정의 대신 새 메소드 추가
- 운이 없게도 하위 클래스에 추가한 메소드와 시그니처가 같고 반환 타입은 다르면 컴파일조차 불가하게 된다. 

<br>

## 상속의 문제 해결책 : 컴포지션(composion)
- 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻
- 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다.

### 전달(forwarding)
- 새 클래스의 인스턴스 메소드들은(private 필드로 참조하는) 기존 클래스의 대응하는 메소드를 호출해 그 결과를 반환한다. 
- 전달 메소드(forwarding method) : 새 클래스의 메소드


```java
// 재사용할 수 있는 전달 클래스 
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```

```java
//상속 대신 컴포지션을 사용했다
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```

- InstrumentedSet를 래퍼 클래스라 한다.
> - 래퍼 클래스는 Set 인스턴스를 감싸고 있다는 뜻에서 나온 이름이다.
> - Decorator pattern의 예시기도 하다.
- 위임(Delegation) : 컴포지션과 전달의 조합, 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우를 의미한다. 

<br>

## 래퍼 클래스의 단점
- 콜백(callback) 프레임워크와는 어울리지 않는다. 

### SELF 문제
- 콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 한다. 
- 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 자신(this) 참조를 넘기고 래퍼 참조가 아닌 내부 객체를 호출하게 된다.

### 참고 사항
- 전달 메소드나 래퍼 객체가 주는 성능상 문제는 큰 영향이 없다고 밝혀졌다. 
- 전달 메소드 작성이 귀찮으면 ForwardingSet 같은 전달 클래스를 인터페이스당 하나씩만 만들어 원하는 기능을 덧씌우는 전달 클래스들을 아주 손쉽게 구현할 수 있다.
> - Guava는 모든 컬렉션 인터페이스용 전달 메소드를 모두 구현해두었다. 

<br>

## 상속과 컴포지션
- 상속은 반드시 하위 클래스가 상위 클래스의 진짜 하위 타입인 상황에서만 사용해야 한다. 
> - `B` is a `A` 관계 일때만 클래스 B는 클래스 A를 상속할 수 있다. 
- 컴포지션을 사용해야 할 상황에서 상속을 사용하는 것은 내부 구현을 불필요하게 노출하는 것이다.
- API가 내부 구현에 묶이고 그 클래스 성능도 영원히 제한되게 된다. 
- 가장 심각한 문제는 클라이언트가 노출된 내부에 직접 접근할 수 있다. 

<br>

### 상속을 결정하는 질문
- 확장하려는 클래스의 API에 아무런 결함이 없는가?
- 결함이 있다면, 이 결함이 여러분 클래스 API 까지 전파돼도 되는가?
> - 컴포지션으로 결함을 숨기는 API를 설계할 수 있지만, 상속은 그 결함까지도 그대로 가져간다.

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
