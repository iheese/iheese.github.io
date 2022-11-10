---
layout: post
title: 'Spring, JPA Interview 대비'
subtitle: 'Spring DI, IOC, AOP, JPA'
date: 2022-11-08 12:00:00 +0900
categories: 'ETC'
background: '/img/posts/etc/git.png'
---

#### Spring DI/IoC는 어떻게 동작하나요?
- IoC(Inversion of Control), 제어의 역전은 프로그램의 제어 흐름을 직접하는 것이 아니라 외부에서 관리하는 것으로 코드의 최종호출은 개발자가  제어하는 것이 아닌 프레임워크 내부에서 결정된 대로 이루어지는 것을 말한다.
- DI(Dependency Injection), 의존성 주입은 Spring 프레임워크에서 지원하는 IoC의 형태로 클래스 사이의 의존 관계를 빈 설정 정보를 바탕으로 컨테이너가 자동으로 연결해줍니다. 

<br>

#### Spring AOP는 어떻게 동작하나요?
- AOP는 Aspect Oriented Programming 약자로, 관점 지향 프로그래밍이라 한다.
- 주요 관심사 로직과 반복되는 부가 로직을 구분하여 처리하는 것을 의미한다.
- Spring AOP는 기본적으로 프록시 패턴을 이용한다. 프록시 패턴이란 어떤 객체를 직접적으로 참조하는 것이 아니라 대행하는 객체를 통해 대상 객체에 접근하는 방식을 말한다.
- AOP 에서는 프록시 객체가 아닌 대상 객체에 직접 접근하게 되면 AOP를 적용하지 않았을 때와 동일한 문제가 생기게 된다. 반복된 코드가 실행되고 유지보수성이 떨어지게 된다. 

<br>

#### DI 종류는 어떤것이 있고, 이들의 차이는 무엇인가요?
- 생성자 주입 : 생성자 호출 시점에 딱 1번만 호출되고 불변, 필수 의존 관계에 사용됩니다.
> - 생성자 주입을 권장하는 이유
> > - 순환 참조를 방지
> > - 불변성을 가짐
> > - 테스트에 용이
- Setter 주입 : 선택, 변경 가능성이 있는 의존 관계에 사용되고 스프링 빈을 선택적으로 등록이 가능합니다.
- 필드 주입 : `@Autowired` 를 사용하고 방법이 간격하여 많이 이용된다. 순환 참조 발생 가능성, 테스트가 불편하다는 단점이 있습니다.

<br>

#### Spring Bean이란 무엇인가요?
- IoC 컨테이너 안에 들어있는 객체로 필요할 때 IoC 컨테이너에서 가져와서 사용합니다. `@Bean`을 사용하거나 xml 설정을 통해 bean으로 등록할 수 있습니다.

<br>

#### 스프링 Bean의 생성 과정을 설명해주세요.
- 객체 생성 - 의존설정 - 초기화 - 사용 - 소멸 과정의 생성 주기를 가지고 있다.
- Spring 컨테이너에 의해 관리되며 ComponentScan을 이용하거나, 직접 @Bean을 등록할 수 있습니다.

<br>

### 스프링 Bean의 Scope에 대해서 설명해주세요.
- 스프링 Bean 스코프는 싱글톤, 프로토타입, request, session, application 등이 있습니다.
> - 싱글톤 : 기본 스코프로 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프를 말합니다.
> - 프로토타입 : 빈의 생성과 의존관계 주입까지만 관여하고 더 이상 관여하지 않는 매우 짧은 범위의 스코프입니다.
> - request : 웹 요청이 들어오고 나갈 때가지 유지하는 스코프입니다.
> - session : 웹 세션이 생성, 종료할 때까지 유지하는 스코프입니다.(시간 설정 가능)
> - application : 웹 서블릿 컨텍스트와 같은 범위로 유지하는 스코프입니다. 

<br>

#### IoC 컨테이너의 역할은 무엇이 있을까요?0
- 어플리케이션 실행이 진행되면서 빈을 인스턴스화하여 DI해주는 역할입니다. 

<br>

#### Autowiring 과정에 대해서 설명해주세요.
- DI의 필드 주입 방식이다. IoC 컨테이너는 빈을 인스턴스화한 뒤, 타입 기반 검색인 리플렉션을 통해 수행되어 빈 객체를 주입해준다. 

<br>

#### Spring Web MVC의 Dispatcher Servlet의 동작 원리에 대해서 간단히 설명해주세요.

<br>

## JPA

#### JPA를 쓴다면 그 이유에 대해서 설명해주세요.
- JPA(Java Persistence API), Java에서 ORM(Object - Relational Mapping) 기술 표준으로 사용되는 인터페이스 모음
> - ORM : 객체와 관계형 데이터베이스의 데이터를 자동으로 매핑해주는 것을 의미한다. 
- JPA의 구현체 Hibernate, EclipseLink, DataNucleus가 있으며 Spring Data JPA는 JPA에 대한 데이터 접근의 추상화라 할 수 있다. GenericDao라는 커스텀 구현체를 제공한다. Hibernate 구현을 기반으로 동작하고 있다. 
- JPA를 사용하면 내부에서 JDBC API를 사용하여 SQL을 호출하여 DB와 통신한다. 
- 장점
> - 쿼리를 일일히 작성하지 않아도 되고 SQL 중심 개발에서 객체 지향적인 개발이 가능해진다.(개발자도 서비스 로직에 더 신경쓸 수 있게 된다.)
> - 생산성 증가
> - 유지보수 용이, 가독성 좋음
> - JPQL로 SQL을 추상화하기 때문에 RDBMS Vendor에 관계없이 동일한 쿼리를 예상할 수 있다.

<br>

#### JPA 영속성 컨텍스트의 이점(5가지)을 설명해주세요.
- 영속성 컨텍스트는 엔티티를 영구 저장하는 환경을 의미한다.
- 1차 캐시 : 조회가 가능하며 1차 캐시에 없으면 DB에 조회하여 1차 캐시에 올려놓는다.
- 동일성 보장 : 동일성 비교가 가능하다.(==)
- 트랜잭션을 지원하는 쓰기 지연 : 트랜잭션을 지원하는 쓰기 지연이 가능하며 트랜잭션 커밋하기 전까지 SQL을 바로 보내지 않고 모아서 보낼 수 있습니다. 
- 변경 감지(Dirty checking) : 1차 캐시에 들어온 데이터를 찍습니다.  commit 되는 시점에 Entity와 스냅샷을 비교하여 update SQL을 생성합니다.
- 지연 로딩 :  엔티티에서 해당 엔티티를 불러올 때 그 때 SQL을 날려 해당 데이터를 가져옵니다. 

<br>

#### @Transactional 움직이는 방식?
- Spring AOP은 JDK Proxy, Spring Boot는 CGLib Proxy를 기본으로 한다.
- JDK Proxy는 타겟의 상위 인터페이스를 상속 받아 프록시를 만든다. 인터페이스를 구현한 클래스가 아니면 의존할 수 없다. 
> - JDK Proxy는 내부적으로 Reflection을 사용하여 추가적인 비용이 든다.(Reflection : 클래스 타입을 몰라도 클래스에 접근하게 해주는 API)
> - 그래서 서비스 인터페이스를 만들고 Impl로 구현하는 관습이 있다.
- CGLib Proxy는 타겟 클래스를 상속 받아 프록시를 만든다. 바이트 코드 조작을 통해 프록시 객체를 생성한다. 

- @Transactional 은 프록시 형태로 동작한다. 어떤 객체에 직접적으로가 아닌 프록시 객체(대행 객체)를 통해 접근한다.
> -  private method에 적용이 불가하고, 무조건 진입점의 Transaction 기준으로 동작한다. 

<br>

#### JPA Propagation 전파단계를 설명해주세요.
- JPA Propagation은 트랜잭션 동작 도중 다른 트랜잭션을 호출하는 상황에 선택할 수 있는 옵션입니다. 
- Default 값은 REQUIRED이며 부모 트랜잭션 안에서 실행되며 부모 트랜잭션이 없을 경우 새로운 트랜잭션을 생성합니다.
- 이외의 값 REQUIRES_NEW, SUPPORTS, MANDATORY, NOT_SUPPORT, NEVER, NESTED 

<BR>

#### N + 1 쿼리 문제는 무엇이고 이것이 발생하는 이유와 이를 해결하는 방법을 설명해주세요.
- 1번 쿼리를 날렸을 때 의도치 않게 N개의 쿼리가 추가적으로 실행되는 것이다.
- 원인
> - JPA Repository로 find 실행시 첫 쿼리에서 하위 엔티티까지 한 번에 가져오지 않고, 하위 엔티티를 사용할 때 추가로 조회하기 때문이다.
> - JPQL에서는 글로벌 페치 전략을 완전히 무시하고 SQL을 생성한다.
- 즉시 로딩에서 발생하는 이유는 JPQL을 사용하는 경우 전체 조회를 했을 때  해당 연관 관계인 하위 엔티티들을 추가 조회하기 때문이다. 
- 지연 로딩에서 발생하는 이유는 지연 로딩 전략을 사용한 하위 엔티티를 로드할 때 JPA에서 프록시 엔티티를 unproxy할때 해당 엔티티를 조회하기 위한 추가적인 쿼리가 발생한다. (하위 엔티티에 접근할 때)
- 해결방법
> - 지연 로딩을 주로 사용하고, JPQL의 JOIN FETCH, @EntityGraph, @Fetch(FetchMode.SUBSELECT), @BatchSize 조절하는 방법이 있다. 


<br>

#### 더 학습할 내용

프론트 컨트롤러 패턴이란 무엇인가요?

Servlet Filter와 Spring Interceptor의 차이는 무엇인가요?

Spring에서 CORS 에러를 해결하기 위한 방법을 설명해주세요.

Bean/Component 어노테이션에 대해서 설명해주시고, 둘의 차이점에 대해 설명해주세요.

POJO란 무엇인가요? Spring Framework에서 POJO는 무엇이 될 수 있을까요?

Spring Web MVC에서 요청 마다 Thread가 생성되어 Controller를 통해 요청을 수행할텐데, 어떻게 1개의 Controller만 생성될 수 있을까요?

Filter는 Servlet의 스펙이고, Interceptor는 Spring MVC의 스펙입니다. Spring Application에서 Filter와 Interceptor를 통해 예외를 처리할 경우 어떻게 해야 할까요?

Spring Application을 구동할 때 메서드를 실행시키는 방법에 대해 설명해주세요.

의존성과 설정값을 생성자 인자로 주입해야 하는 이유에 대해 설명해주세요.




Reference:
- [Backend-Interview-Question_ ksundong](https://github.com/ksundong/backend-interview-question)
