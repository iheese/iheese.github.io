---
layout: post
title: '[EFFECTIVE JAVA] ITEM 11, equals를 재정의하려거든 hashCode도 재정의하라'
subtitle: ''
date: 2023-06-07 16:00:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 11, equals를 재정의하려거든 hashCode도 재정의하라

<br>

## equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.
- 재정의하지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

<br>

## Object 명세의 규약

- equals 비교에 사용되는 값이 변경되지 않았다면 어플리케이션이 실행되는 동안은 일관되게 같은 값을 반환해야 한다.
> - 어플리케이션이 다시 실행된다면 값이 달라져도 된다.
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 다르다고 판단해도 두 객체의 hashCode가 서로 다른 값을 반환할 필요없다.
> - 다만 다른 객체에 대해서 다른 값을 반환해야 해시테이블의 성능이 좋아진다.  

<br>

## 좋은 해시 함수
- 논리적으로 같은 객체는 같은 hashCode를 반환해야 한다. 
- 서로 다른 인스턴스에는 다른 hashCode를 반환해야 한다.
- 서로 다른 인스턴스들이 32비트 정수 범위에 균일하게 분배되게 해야 한다.

<br>

## 좋은 hashCode 작성법

#### 1 . int 변수 result를 선언하고 값 c 로 초기화 한다.

- c는 해당 객체의 첫 번째 핵심 필드를 계산한 해시코드
> - 핵심 필드는 equals 비교에 사용되는 필드

#### 2 . 객체의 나머지 핵심 필드에 대해 계산을 수행한다.

a . 해당 필드의 해시 코드를 계산한다

- 기본 타입 필드 :  `Type.hashCode(f)` , EX) `Integer.hashCode(f)`
- 참조 타입 필드 + equals 가 재귀적으로 호출 비교 : hashCode 재귀적으로 호출
> - 복잡해질 것 같으면 필드의 표준형(canonical representation)을 만들어 그 표준형의 hashCode를 호출
> - 필드의 값이 null이면 0사용 (전통적으로)
- 배열 타입 필드 : 핵심 원소 각각을 별도 필드처럼 다룬다. 
> - 배열에 핵심 원소가 하나도 없다면 상수(0 추천) 사용한다.
> - 모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용한다. 

b . 2.a 단계에서 계산한 해시코드 c로 result를 갱신한다.
- result - 31 * result + c;

#### 3 . result를 반환한다. 

<br>

## hashCode 구현 후 확인해볼 점

#### 동치인 인스턴스에 대해 똑같은 해시코드를 반환할지 자문해보자
- 직관을 검증할 단위 테스트를 작성해보자
> - equals, hashCode를 AutoValue로 생성했다면 패스

<br>

#### equals 비교에 사용되지 않는 파생 필드는 해시코드 계산에서 제외하자

```java
   // 전형적인 hashCode 메서드 
   @Override public int hashCode() {
       int result = Short.hashCode(areaCode);  // 31 * result 필드를 곱하는 순서에 따라 result 값이 달라진다. 
       result = 31 * result + Short.hashCode(prefix); //31은 전통적으로 소수를 사용
       result = 31 * result + Short.hashCode(lineNum);
       return result;
   }
```

<br>

#### 성능을 위해 해시코드 계산시 핵심 필드를 생략하지 말자
- 오히려 해시 품질이 나빠져 해시테이블 성능을 심각하게 떨어뜨릴 수도 있다.
- 어떤 필드는 특정 역역에 몰린 인스턴스를 넓은 범위로 퍼뜨리는 효과가 있을 수도 있다. 

<br>

####  hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자
- 클라이언트가 해당 값에 의지하지 않게 된다.
- 추후에 계산 방법이 바뀔 수도 있기 때문이다. 

<br>

## hashCode 의 다른 방법들

- 해시 충돌이 더 적은 방법 : 구아바의 [com.google.common.hash.Hashing](https://guava.dev/releases/21.0/api/docs/com/google/common/hash/Hashing.html)을 참고
- Object 클래스의 hash() 메소드

```java
@Override public int hashCode(){
	return Objects.hash(lineNum, prefix, areaCode);
}
```

- hashCode 함수를 한 줄로 작성할 수 있다.
- 속도는 더 느린 편이다
> - 입력 인수를 위한 배열이 만들어지고
> - 기본 타입이 있으면 박싱, 언박싱을 거쳐야 한다.
- 성능에 민감하지 않을 때 추천

<br>

##  해시코드 캐싱 방법

- 클래스가 불변, 해시코드를 계산하는 비용이 크다면 > 캐싱 방법 고려

```java
   private int hashCode; // 자동으로 0으로 초기화된다.

   @Override public int hashCode() {
       int result = hashCode;
       if (result == 0) {
           result = Short.hashCode(areaCode);
           result = 31 * result + Short.hashCode(prefix);
           result = 31 * result + Short.hashCode(lineNum);
           hashCode = result;
       }
       return result;
   }
```

- 객체가 주로 해시의 키로 사용될 것 같은 경우 > 인스턴스가 만들어질 때 해시코드를 계산
- 객체가 해시의 키로 사용되지 않는 경우 > 지연 초기화(lazy initialization) 전략 (hashCode가 처음 불릴 때 계산)
> - 스레드 안정성 고려 필요


<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)