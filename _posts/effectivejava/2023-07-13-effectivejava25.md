---
layout: post
title: '[EFFECTIVE JAVA] ITEM 25, 톱레벨 클래스는 한 파일에 하나만 담으라'
subtitle: ''
date: 2023-07-13 12:00:00 +0900
categories: [effectivejava]
background: '/img/posts/effectiveJava/effectiveJava.jpg'
---

# ITEM 25, 톱레벨 클래스는 한 파일에 하나만 담으라

<br>

## 톱레벨 클래스
- 우리가 일반적으로 써왔던 클래스, 내부 클래스와 구분하기 위해 톱레벨 클래스라고 한다.
- 소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자 컴파일러는 불평하지 않는다.
- 하지만 이는 아무런 득이 없으며 심각한 위험을 감수해야 한다. 

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```


```java
// Utensil.java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}

```

- 두 클래스가 한 파일(Utensil.java)에 정의되었다. 


```java
// Dessert.java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

- 두 클래스가 한 파일(Dessert.java)에 정의되었다. 

- Main.java를 컴파일 > Utensil.java 확인 > Dessert.java 에러 : 같은 클래스 정의 존재 확인

- 컴파일러에게 어떤 소스 파일을 먼저 주냐에 따라 pancake나 potpie로 출력 결과가 달라진다. 

<br>

## 해결책

- Utensil.java와 Dessert.java 소스 파일을 분리한다. 
- 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용한다. 

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
 