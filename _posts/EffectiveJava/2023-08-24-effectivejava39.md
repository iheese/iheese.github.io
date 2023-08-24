---
layout: post
title: '[EFFECTIVE JAVA] ITEM 39, 명명 패턴보다 애너테이션을 사용하라'
subtitle: ''
date: 2023-08-24 15:40:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/effectiveJava/effectiveJava.jpg'
published : true
---

# ITEM 39, 명명 패턴보다 애너테이션을 사용하라

<br>

## 명명 패턴의 단점
> - EX) JUnit3 까지 테스트 메서드 이름을 test로 시작하게끔 하였다. 
- 오타가 나면 안된다.
> - 메소드 이름을 다른 것으로 지으면 JUnit3는 이를 지나치기 때문에 통과했다고 오해할 수 있다.
- 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다. 
> - 클래스 이름을 다른 것으로 지으면 테스트가 전혀 수행되지 않는다.
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.
> - 특정 예외를 던저야 하는 테스트는 방법이 없다. 

<br>

## 마커(marker) 애너테이션

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

- 메타 에너테이션(meta-annotation) : 애너테이션 선언에 다는 애너테이션
> - `@Retention(RetentionPolicy.RUNTIME)` : @Test 가 런타임에도 유지되어야 한다는 표시
> - `@Target(ElementType.METHOD)` : @Test가 메소드 선언에만 사용되어야 한다는 표시
- 마커 애너테이션(marker-annotation) : 아무 매개변수 없이 단순히 대상에 마킹한다는 뜻이다.  
- 클래스에 영향을 주진 않지만 관심 있는 프로그램에게 추가 정보를 제공할 뿐이다. 
- 대상 코드의 의미는 그대로 둔 채 애너테이션에 관심 있는 도구에서 특별한 처리를 할 기회를 준다.

<br>

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n",
                passed, tests - passed);
    }
}
```

- 마커 애너테이션을 처리하는 프로그램
- 리플렉션을 사용해 @Test 애너테이션이 달린 메소드를 찾는다.
- 예외에 담긴 실패 정보를 추출해(getCause) 출력한다. 

<br>

## 매개 변수를 가진 애너테이션
- 예외를 던져야 성공하는 테스트 만들기

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}
```

- `Class<? extends Throwable>` : Throwable을 확장한 클래스의 Class 객체
> - 모든 예외와 오류 타입을 다 수용한다. 
> - EX) `@Test(value = "RuntimeException.class")`

<br>

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (InvocationTargetException wrappedEx) {
        Throwable exc = wrappedEx.getCause();
        Class<? extends Throwable> excType =
                m.getAnnotation(ExceptionTest.class).value();
        if (excType.isInstance(exc)) {
            passed++;
        } else {
            System.out.printf(
                    "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                    m, excType.getName(), exc);
        }
    } catch (Exception exc) {
        System.out.println("잘못 사용한 @ExceptionTest: " + m);
    }
}
```

- 위처럼 main 메소드를 수정한다.
- 애너테이션 매개변수의 값을 추출하여 올바른 예외를 던지는지 확인하는데 사용한다. 
> - 테스트 프로그램이 문제없이 컴파일되면 애너테이션 매개변수가 가리키는 예외가 올바른 타입이라는 뜻이다.
> - 컴파일 타임에는 존재했으나 런타임에는 존재하지 않을 수 있다. 이 때는 TypeNotPresentException을 던진다.

<br>

## 배열 매개변수를 받는 애너테이션 타입

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```

```java
@ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})
```

- 위처럼 사용한다. 

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (Throwable wrappedExc) {
    Throwable exc = wrappedExc.getCause();
    int oldPassed = passed;
    Class<? extends Throwable>[] excTypes =
      m.getAnnotation(ExceptionTest.class).value();
    for (Class<? extends Throwable> excType : excTypes) {
      if (excType.isInstance(exc)) {
        passed++;
        break;
      }
    }
    if (passed == oldPassed)
      System.out.printf("테스트 %s 실패: %s %n", m, exc);
  }
}
```

- for문을 이용해서 처리한다.

<br>

## Repeatable
- Java 8 부터 지원한다.
- 배열 매개변수를 사용하는 대신 애너테이션에 `@Repeatable` 메타애너테이션을 다는 방식이다. 
- 하나의 프로그램 요소에 여러 번 달 수 있다.

### 사용법
- `@Repeatable` 을 단 애너테이션을 반환하는 컨테이너 애너테이션을 하나 더 정의한다.
- `@Repeatable` 에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
- 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메소드를 정의해야 한다.
- 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention), 적용 대상(@Target)을 명시해야 한다.

```java
// 반복 가능 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}

// 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] value();
}
```


```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {
```

- 위는 적용 방법이다.

<br>

### 주의할 점
- 여러 개 달면 하나만 달았을 때와 구분하기 위해 해당 컨테이너 애너테이션 타입이 적용된다.
- getAnnotationsByType 메소드는 반복 가능 애너테이션과 컨테이너 애너테이션 둘을 구분하지 않는다. 
- isAnnotationPresent 메소드는 반복 가능 애너테이션이 달렸는지 검사한다.(true : 반복 가능 애너테이션)
- 달려 있는 수와 상관없이 모두 검사하려면 둘을 따로 따로 확인해야 한다. 

```java
f (m.isAnnotationPresent(ExceptionTest.class)
    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (Throwable wrappedExc) {
    Throwable exc = wrappedExc.getCause();
    int oldPassed = passed;
    ExceptionTest[] excTests =
      m.getAnnotationsByType(ExceptionTest.class);
    for (ExceptionTest excTest : excTests) {
      if (excTest.value().isInstance(exc)) {
        passed++;
        break;
      }
    }
    if (passed == oldPassed)
      System.out.printf("테스트 %s 실패: %s %n", m, exc);
  }
}
```

- 애너테이션을 여러 번 달 때의 코드 가독성을 높였다.
- 애너테이션을 선언하고 이를 처리하는 부분에서 코드양이 늘어난다.
- 처리 코드가 복잡해져 오류가 날 가능성이 커질 수 있다. 

<br>

## 정리
- 애너테이션이 명명 패턴보다 낫다.
- 다른 프로그래머가 소스코드에 추가 정보를 제공할 수 있는 도구를 만드는 일을 한다면 적당한 애너테이션 타입도 함께 제공하자.
- 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.
- 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입을 사용해야 한다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
