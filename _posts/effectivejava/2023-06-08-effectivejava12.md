---
layout: post
title: '[EFFECTIVE JAVA] ITEM 12, toString을 항상 재정의하라'
subtitle: ''
date: 2023-06-08 12:00:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 12, toString을 항상 재정의하라

<br>

- Object.toString 메소드 : 클래스_이름@16진수로_표시한_해시코드
> - EX) PhoneNumber@adbbd

<br>

## toString 메소드의 규약

- '간결하면서 읽기 쉬운 형태의 유익한 정보'를 반환해야 한다.
- 모든 하위 클래스에서 이 메소드를 재정의하라
> - 사용하기에 훨씬 좋고, 디버깅하기 용이해진다.

- 객체를 println, printf, 문자열 연경 연산자(+), assert 구문에 넘길 때, 혹은 디버거가 객체를 출력할 때 자동으로 불린다. 
- 우리가 직접 호출하지 않더라도 어딘가에서 쓰일 것이다.

- EX) `System.out.println(phoneNumber + "에 연결할 수 없습니다.")`

- 실전에서는 객체가 가진 주요 정보 모두를 반환하는 것이 좋다. (거대하다면 요약 정보)

<br>

## toString 메소드 반환값의 포맷 문서화

- 값 클래스(예: 전화번호, 행렬)는 문서화를 권한다.
- 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩토리나 생성자를 제공해주면 좋다.
> - `BigInteger, BigDecimal`

- 장점
> - 표준적, 명확하고, 사람이 읽을 수 있게 된다.
> - 값 입출력에 사용 가능, csv 파일처럼 사람이 읽을 수 있는 데이터 객체로 저장 가능

- 단점
> - 포맷을 한 번 명시하면 평생 그 포맷에 얽매이게 된다. 
> - `>` 개발자들이 그 포맷에 맞춰 파싱, 새로운 객체 생성, 영속 데이터로 저장하는 코드 작성 
> - `>` 향후 포맷이 바뀌면 데이터, 코드 엉망
> - `>` 개발자들 절규
> > - 포맷이 명시되지 않으면 정보를 더 넣거나 포맷을 개선할 수 있는 유연성을 보장할 수 있다. 

<br>

### 포맷을 명시하든 아니든 우리의 의도를 명확히 밝혀야 한다.

- 포맷를 명시하기로 했다면

```java
    /**
     * 이 전화번호의 문자열 표현을 반환한다.
     * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
     * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
     * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
     *
     * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
     * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
     * 전화번호의 마지막 네 문자는 "0123"이 된다.
     */
   @Override public String toString() {
       return String.format("%03d-%03d-%04d",
               areaCode, prefix, lineNum);
   }
```

- or 포맷을 명시하지 않기로 했다면

```java
    /**
     * 이 전화번호의 문자열 표현을 반환한다.
     * 다음은 이 설명의 일반적인 형태이나,
     * 추후에 변경될 수 있다. 
     *
     * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
     */
   @Override public String toString() {...}
```

<br>

### 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API 를 제공하자
- getter 같은 메소드를 의미하는 것 같다.
- toString 반환값을 파싱하는 작업은 성능을 나쁘게 하고 필요하지 않은 작업이다.
> - 접근자를 제공하지 않으면 이 포맷이 준 표준 API 역할을 하게 된다. 

<br>

## toString 메소드를 제공할 이유
- 정적 유릴리티 클래스, 열거타입은 이미 완벽한 toString을 제공하니 재정의 필요하지 않다. 
- 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스는 toString 재정의가 필요하다.
> - 대다수의 컬렉션 구현체는 추상 컬렉션 클래스의 toString 메소드를 상속해 사용한다. 

<br>

## 정리
- 모든 구체 클래스에서 Object의 toString을 재정의하자
> - 상위 클래스에서 알맞게 정의된 경우 예외
- 사용하기 편하고 디버깅을 쉽게 해준다
- toString은 해당 객체에 관해 명확하고 유용한 정보를 읽기 좋게 반환해야 한다. 

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)