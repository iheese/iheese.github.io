---
layout: post
title: '[EFFECTIVE JAVA] ITEM 24, 멤버 클래스는 되도록 static으로 만들라'
subtitle: ''
date: 2023-07-13 14:00:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
---

# ITEM 24, 멤버 클래스는 되도록 static으로 만들라

<br>

## 중첩 클래스(nested class)
- 다른 클래스 안에 정의된 클래스
- 자신을 감싼 바깥 클래스에서만 쓰여야 하고, 톱레벨 클래스로 만들어야 한다.

### 중첩 클래스의 종류
- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스

<br>

### 정적 멤버 클래스
- 다른 클래스 안에서 선언된다.
- 바깥 클래스의 private 멤버에도 접근 가능하고 다른 정적 멤버와 같은 접근 규칙을 적용받는다.
- 흔히 바깥 클래스와 함꼐 쓰일 때 유용한 public 도우미 클래스로 쓰인다. 

```java
public class Job {
    private String name;

    // 열거 타입도 암시적 static
    public enum Jobs {
        STUDENT, TEACHER, POLICE, DESIGNER
    }

    public static class publicClass {
        private int age;

        public void method() {
            Job outerClass = new Job();
            System.out.println(outerClass.name); // 메소드가 public이므로 접근 가능하다.
        }
    }

    private static class privateClass {
         private int age;

        public void method() {
            Job outerClass = new Job();
            System.out.println(outerClass.name);  // 메소드가 private이므로 접근 불가하다.
        }
    }
}
```

<br>

### (비정적) 멤버 클래스
- 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다. 
- `바깥 클래스명.this` 으로 바깥 클래스 메소드, 참조를 호출할 수 있다.

#### 주로 어댑터의 역할을 한다.
- 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용한다.
- EX) Map 인터페이스의 구현체: 자신의 컬렉션 뷰를 구현할 때 사용, 다른 컬렉션 인터페이스 구현 : 자신의 반복자를 구현할 때 사용

```java
public class MySet<E> extends AbstractSet<E> {
    // ...생략

    @Override public Iterator<E> iterator() {
        return new MyIterator();
    }

    private class MyIterator implemnets Iterator<E> {
        // ...
    }
}

```

### 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자
- static을 생략하면 바깥 인스턴스로부터 숨은 외부 참조를 갖게된다.
- 이 관계 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차치하며 생성 시간도 더 걸린다. 
- 더군다나 GC가 바깥 클래스의 인스턴스를 수거하지 못해 메모리 누수가 생긴다.

<br>

### private 정적 멤버 클래스
- 바깥 클래스가 표현하는 객체의 한 부분(구성요소)을 나타낼 때 쓴다.
- 정적 멤버 클래스를 private 으로 선언하면 바깥 클래스에서만 접근이 가능해진다.
- EX) 키와 값을 매핑시키는 Map 인스턴스 : Map 구현체는 엔트리(Entry) 객체를 가지고 있다. 엔트리의 메소드들은 직접 맵을 사용하지 않는다.
> - 엔트리를 비정적 멤버 클래스로 표현하는 것은 낭비, 그러므로 private 정적 멤버 클래스가 알맞다.

<br>

### 익명 클래스
- 이름이 없다.
- 익명 클래스는 바깥 클래스의 멤버도 아니다.
- 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
- 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.
- 정적 문맥에서라도 상수 변수 이외에 정적 멤버는 가질 수 없다.
> - 상수 표현을 위해 초기화된 final 기본 타입과 문자열 필드만 가질 수 있다. 

```java
// 인터페이스 선언
public interface MyInterface {
  // 추상 메서드 선언
  public void do();
}

// 구현 클래스 정의
public class MyClass implements MyInterface {
  @Override
  public void do() {
    System.out.println("MyClass의 do() 메서드 호출");
  }
}
```

### 익명 클래스의 제약사항
- 선언한 지점에서만 인스턴스를 만들 수 있다.
- instanceOf 검사, 클래스 이름이 필요한 검사를 할 수 없다.
- 여러 인터페이스 구현 불가, 구현과 동시에 다른 클래스 상속 불가하다.
- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상속 타입에서 상속한 멤버 외에는 호출할 수 없다.
- 익명 클래스는 표현식 중간에 등장하므로 짧지 않으면 가독성을 해친다.

#### 익명 클래스 사용 예

- 자바7 전에 람다를 지원하기 전에는 작은 함수 객체나 처리 객체를 만드는데 사용하였다. > 자바8 부터 람다가 등장했다. 이제 람다를 쓰자
- 정적 팩터리 메소드를 구현할 떄 사용하기도 한다.

<br>

### 지역 클래스
- 가장 드물게 사용되고 지역변수를 선언할 수 있는 곳이면 실질적으로 어디서든 선언할 수 있다.
- 지역변수와 유효범위가 같다.
- 멤버 클래스처럼 이름이 있고 반복하여 사용할 수 있다.
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.
- 익명 클래스처럼 정적 멤버는 가질 수 없고 가독성을 위해 짧게 작성해야 한다. 

```java
public class Example {
    private int number;

    public Example(int number) {
        this.number = number;
    }

    public void method() {
        // 지역변수처럼 선언해서 사용할 수 있다.
        class LocalClass {
            private String name;

            public LocalClass(String name) {
                this.name = name;
            }

            public void print() {
                // 비정적 문맥에선 바깥 인스턴스를 참조 할 수 있다.
                System.out.println(number + name);
            }
        }

        LocalClass localClass = new LocalClass("local");
        localClass.print();
    }
}
```

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
 