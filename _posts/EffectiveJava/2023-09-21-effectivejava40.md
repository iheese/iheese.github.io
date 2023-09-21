---
layout: post
title: '[EFFECTIVE JAVA] ITEM 40, @Override 애너테이션을 일관되게 사용하라'
subtitle: ''
date: 2023-09-21 13:35:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 40, @Override 애너테이션을 일관되게 사용하라

<br>

## @Override
- 메서드 선언에 달 수 있다.
- 상위 타입의 메소드를 재정의했음을 의미한다. 

<br>

## @Override 을 일관되게 사용하면 여러 가지 악명 높은 버그를 예방해준다.

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```

- 위 코드에서 equals, hashCode 메소드를 재정의(Overriding) 하였으나 equals를 다중 정의(Overloading)해버렸다.
> - Object의 equals 메소드를 재정의하려면 매개변수로 Object를 받아야 한다.
- @Override 애너테이션응 달면 컴파일 오류가 뜬다.

```java
@Override public boolean equals(Object o) {
        if (!(o instanceof Bigram2))
            return false;
        Bigram2 b = (Bigram2) o;
        return b.first == first && b.second == second;
    }
```

- 바로 올바르게 수정할 수 있게 된다. 

<br>

## 상위 클래스의 메소드를 재정의하려는 모든 메소드에 @Override 애너테이션을 달자.

#### 예외
- 구체 클래스에서 상위 클래스의 추상 메소드를 재정의할 때는 굳이 @Override를 달지 않아도 된다. 
> - 구체 클래스인데 아직 구현하지 않은 추상 메소드가 남아 있다면 컴파일러가 그 사실을 바로 알려준다.
- 모든 재정의 메소드에 @Override를 일괄로 붙여두어도 상관은 없다.
- IDE애서 사용하게 잘 지원해주는 설정들이 있다.

<br>

### 추상 클래스, 인터페이스에서 상위 클래스나 상위 인터페이스의 메소드를 재정의하는 모든 메소드에 @Override 를 다는 것이 좋다.
- 인터페이스가 디폴트 메소드를 지원하면서 헷갈릴 수 있으니, 애너테이션을 닮으로써 재정의할 클래스들의 시그니처를 재확인할 수 있다.
- 인터페이스에 디폴트 메소드가 없다면 애너테이션을 떼고 깔끔히 유지해도 좋다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
