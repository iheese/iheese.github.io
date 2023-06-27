---
layout: post
title: '[EFFECTIVE JAVA] ITEM 17, 변경 가능성을 최소화하라'
subtitle: ''
date: 2023-06-27 14:00:00 +0900
categories: 'EffectiveJava'
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 17, 변경 가능성을 최소화하라

<br>

## 불변 클래스
- 그 인스턴스 내부 값을 수정할 수 없는 클래스
- 가변 클래스보다 설계하고 구현하기 쉽고, 오류가 생길 여지도 적고 안전하다.
- EX) String, 기본 타입의 박싱된 클래스들, BigInteger, BingDecimal

<br>

## 불변 클래스 만드는 규칙

#### 객체의 상태를 변경하는 메소드(변경자, setter)를 제공하지 않는다.

#### 클래스를 확장할 수 없도록 한다.
- 하위 클래스에서 나쁜 의도로 상태를 변하게 만드는 사태를 막아준다.

#### 모든 필드를 final로 선언한다.
- 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러내는 방법, 새로 생성된 인스턴스를 동기화없이 다른 스레드로 건네도 문제없이 보장

#### 모든 필드를 private으로 선언한다. 
- 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다.

#### 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
- 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다. 

<br>

## 불변 객체의 장점

- 함수형 프로그래밍에 익숙해지면 불변이 되는 영역의 비율이 높아지는 장점이 있다.

#### 불변 객체는 단순하다.
- 불변 객체는 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다. 

#### 불변 객체는 근복적으로 스레드 안전하여 따로 동기화할 필요없다.
- 불변 객체에 대해서는 그 어떤 스레드도 다른 스레드에 영향을 줄 수 없다.

#### 스레드 안전하기 때문에 불변 객체는 안심하고 공유할 수 있다. 
- 덕분에 불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 생성하지 않게 해주는 정적 팩터리를 제공할 수 있다.
- 불변 객체를 자유롭게 공유할 수 있다는 점은 방어적 복사도 필요없다는 것을 의미한다.

#### 불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다. 

#### 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다. 
- 값이 바뀌지 않는 구성 요소들로 이뤄진 객체라면 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하다.
- EX) 불변 객체는 map의 key와 집합(set)의 원소로 쓰기 안성맞춤이다. 

#### 불변 객체는 그 자체로 실패 원자성을 제공한다.
- 실패 원자성(failure atomicity) : 메소드에서 예외가 발생한 후에도 그 객체는 여전히 유효한(메소드 호출 전과 같은) 상태여야 한다.   
- 상태가 절대 변하지 않으니 불일치 상태가 될 수 없다.

<br>

## 불변 객체의 단점

#### 값이 다르면 반드시 독립된 객체로 반들어야 한다.
- 값의 가짓수가 많다면 이를 모두 만드는 데 큰 비용이 든다. 
- EX) BigInteger의 flipBit 메서드 (O(N)) : 불변 `vs` BitSet의 flip 메서드 (O(1)) : 가변

### 문제 대처법

#### 다단계 연산(multistep operation) 예측하여 기본 기능으로 제공
- 다단계 연산 속도를 높여주는 가변 동반 클래스를 package-private으로 둔다. 

#### 다단계 연산(multistep operation) 예측이 안될 때
- 다단계 연산 속도를 높여주는 가변 동반클래스를 public으로 제공한다. 
- EX) String의 가변 동반 클래스 : StringBuilder(단일스레드, 동기화 고려 필요없을 때), StringBuffer(멀티스레드 환경)

<br>

## 불변 클래스를 만드는 설계 방법
#### final 클래스
- 유연하지 못한 편이다.

#### 모든 생성자를 private, package-private으로 만들고 public 정적 팩터리를 제공하는 방법(ITEM1)

```java
public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
}
```

- 다수의 구현 클래스를 활용한 유연성을 제공

<br>

### BigInteger, BingDecimal 설계 과정
- 두 클래스의 메소드들은 모두 재정의가 가능하게 설계되었다. 하위 호환성 문제
- 인수로 받은 객체가 진짜인지 확인해야한다.
> - 신뢰할 수 없는 하위 클래스의 인스턴스라면 가변이라 가정하고 방어적으로 복사하여 사용해야한다.

```java
public static BigInteger safeInstance(BigInteger val){
    return val.getClass() == BigInteger.class ? val : new BigInteger(val.toByteArray());
}
```

<br>

### 불변 객체 기준 완화 for 성능
- "어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없다."
- 계산 비용이 큰 값을 처음 쓰일 때 계산하여 final이 아닌 필드에 캐싱해둔다.
- EX) String 클래스는 불변객체지만, final이 아닌 필드 hash를 이용해 캐시하여 hashCode 재연산 비용을 줄인다.

<br>

## 정리

#### 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
- getter가 있다고 해서 무조건 setter를 만들지 말자

#### 무거운 값 객체도 불변으로 만들 수 있는지 고심해야 한다.
- 성능 때문에 어쩔 수 없다면 가변 동반 클래스를 public 클래스로 제공하자

#### 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자
- 객체를 예측하기 쉬워지고 오류 가능성을 줄일 수 있다.

#### 다른 합당한 이유가 없다면 private final이어야 한다.

#### 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다. 


<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)
