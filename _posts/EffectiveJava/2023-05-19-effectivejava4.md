---
layout: post
title: '[EFFECTIVE JAVA] ITEM 4, 인스턴스화를 막으려거든 private 생성자를 사용하라'
subtitle: ''
date: 2023-05-19 10:15:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 4, 인스턴스화를 막으려거든 private 생성자를 사용하라

<br>

## 정적 메소드와 정적 필드만 담은 클래스의 쓰임새
- 해당 클래스들은 인스턴스로 만들어서 설계한 클래스가 아님
- 기본 타입 값이나 배열 관련 메서드들을 모아 놓을 때
> - [`java.lang.Math`](https://docs.oracle.com/javase/8/docs/api/java/lang/Math.html)
> - [`java.util.Arrays`](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html)
- 특정 인터페이스를 구현하는 객체를 생성해주는 정적메서드(혹은 팩터리)를 모아놓을 수 있다.
> - [`java.util.Collections`](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html)
- final 클래스와 관련한 메서드를 모아 놓을 떄
> - final 클래스는 상속이 불가능하므로 하위 클래스를 만들어 메서드를 넣을 수 없다. 

<br>

## 인스턴스화를 막는 방법
- 추상 클래스로 만드는 것은 인스턴스화를 막을 수 없다. > 오히려 상속하여 사용하라는 것으로 오해 가능
- 생성자 명시하지 않으면 자동으로 public 기본 생성자(매개변수가 없는 생성자)가 생성된다. > 인스턴스화가 가능해짐

### 해결법
- private 생성자를 추가하여 클래스의 인스턴스화를 막자

```java
public class UtilityClass {
    // 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용).
    private UtilityClass() {
        throw new AssertionError();
    }
    // 나머지 코드는 생략
}
```

- 위 코드처럼 주석을 통해 적절한 주석을 통해 설명을 붙여주자
- 상속을 불가능하게 하는 효과가 있다. 
> - 모든 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출한다. 클래스의 생성자를 private으로 선언했으니 하위 클래스가 상위 클래스에 접근 할 수 없게 된다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 깃헙](https://github.com/WegraLee/effective-java-3e-source-code)