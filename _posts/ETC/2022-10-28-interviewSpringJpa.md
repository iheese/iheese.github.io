---
layout: post
title: 'Spring, JPA Interview 대비'
subtitle: 'Spring DI, IOC, AOP, JPA'
date: 2023-07-04 09:00:00 +0900
categories: 'ETC'
background: '/img/posts/etc/git.png'
---

#### Spring DI/IoC/DL는 어떻게 동작하나요?
- IoC(Inversion of Control) :  제어의 역전은 프로그램의 제어 흐름을 직접하는 것이 아니라 외부에서 관리하는 것으로 코드의 최종호출은 개발자가 제어하는 것이 아닌 프레임워크 내부에서 결정된 대로 이루어지는 것을 말한다.
- DI(Dependency Injection) :  의존성 주입은 Spring 프레임워크에서 지원하는 IoC의 형태로 클래스 사이의 의존 관계를 빈 설정 정보를 바탕으로 컨테이너가 자동으로 연결해줍니다.
- DL(Dependency Lookup) : 의존대상(사용할 객체)를 검색(lookup)을 통해 반환받는 방식 `factory.getBean(id)`, 컨테이너와 종속성이 생김.

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
> - 테스트가 불편한 이유는 DI 컨테이너를 띄우는 코드를 작성해야 하기 때문이다. 그 외 방법들은 new 생성자를 통해 간편하게 테스트 코드를 작성할 수 있다. 

<br>

#### Spring Bean이란 무엇인가요?
- 스프링 컨테이너에 의해 관리되는 재사용 가능한 소프트웨어 컨포넌트
- IoC 컨테이너 안에 들어있는 객체로 필요할 때 IoC 컨테이너에서 가져와서 사용합니다. `@Bean`을 사용하거나 xml 설정을 통해 bean으로 등록할 수 있습니다.

<br>

#### 스프링 Bean의 생성 과정을 설명해주세요.
- 객체 생성 - 의존설정 - 초기화 - 사용 - 소멸 과정의 생성 주기를 가지고 있다.
- Spring 컨테이너에 의해 관리되며 ComponentScan을 이용하거나, 직접 @Bean을 등록할 수 있습니다.

<br>

### 스프링 Bean의 Scope에 대해서 설명해주세요.
- 스프링 Bean 스코프
> - 싱글톤 : 기본 스코프로 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 기본이 되는 스코프를 말합니다.
> > - 컨테이너 생성시 생성, 초기화 / 싱글톤 스코프 빈을 조회하면 항상 같은 인스턴스 리턴
> - 프로토타입 : 빈의 생성과 의존관계 주입까지만 관여하고 더 이상 관여하지 않는 매우 짧은 범위의 스코프입니다.
> > - 빈 조회시 생성, 초기화, 이후 관리x, 프로토타입 빈을 받은 클라이언트가 객체를 관리해야 함
> > - 스프링 빈을 여러 곳에서 사용하려하는데 충돌이 날 때 사용하면 좋을 것 같다. 
- 웹 관련 스코프
> - request : 웹 요청이 들어오고 나갈 때가지 유지하는 스코프입니다.
> - session : 웹 세션이 생성, 종료할 때까지 유지하는 스코프입니다.(시간 설정 가능)
> - application : 웹 서블릿 컨텍스트와 같은 범위로 유지하는 스코프입니다. 

<br>

#### IoC 컨테이너의 역할은 무엇이 있을까요?
- 어플리케이션 실행이 진행되면서 빈을 인스턴스화하여 DI해주는 역할입니다. 

<br>

#### Autowiring 과정에 대해서 설명해주세요.
- DI의 필드 주입 방식이다. IoC 컨테이너는 빈을 인스턴스화한 뒤, 타입 기반 검색인 리플렉션을 통해 수행되어 빈 객체를 주입해준다. 

<br>

#### Spring Web MVC의 Dispatcher Servlet의 동작 원리에 대해서 간단히 설명해주세요.
- 디스패처서블릿이 초기화되면서 스프링 컨테이너도 생성됩니다.  

![mvc](https://github.com/iheese/TIL/assets/88040158/4534c49e-9741-4edd-9abc-62a09cfd0ce7)

1. 클라이언트의 요청을 디스패처서블릿이 받습니다.
2. 요청을 컨트롤러로 위임할 핸들러 어댑터를 찾아서 전달합니다.
3. 핸들러 어댑터가 컨트롤러로 요청을 위임합니다.
4. 컨트롤러가 비즈니스 로직을 처리합니다.
5. 컨트롤러가 반환값을 리턴합니다.
6. 핸들러 어댑터가 반환값을 처리합니다.
7. 서버의 응답을 클라이언트에게 리턴합니다. 

<br>

#### Spring Web MVC에서 요청 마다 Thread가 생성되어 Controller를 통해 요청을 수행할텐데, 어떻게 1개의 Controller만 생성될 수 있을까요?
- Controller 클래스의 정보는 JVM Runtime Data Area의 메소드 영역에 저장이 되기 때문에 모든 스레드가 공유할 수 있게 됩니다. 그러므로 블록될  일도 없게 됩니다. 공유된 메소드 영역의 클래스 정보를 가져와서 객체를 생성하면 힙에서 사용하고 GC에 의해 삭제되게 됩니다. 
> - 새로운 요청은 새로운 스레드를 만들어낸다. 
> - 톰캣의 스레드 기본 갯수는 200개
> - 메소드 영역, 힙 영역 스레드 공유가 가능하다.  

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

#### 프론트 컨트롤러 패턴이란 무엇인가요?
- 맨 앞에 하나의 서블릿을 두어 각각의 컨트롤러가 해야했던 공통 영역을 처리하고 알맞는 컨트롤러에 분배해주는 패턴입니다. 
- Spring MVC의 디스패처서블릿이 예입니다. 

<br>

#### Servlet Filter와 Spring Interceptor의 차이는 무엇인가요?
- 필터는 Servlet filter 이며  javax.servlet 스펙에 포함되는 클래스입니다. 디스패처 서블릿에 요청이 전달되기 전/후 URL 패턴에 맞는 모든 요청을 처리할 수 있는 기능을 제공합니다. 즉 스프링 컨텍스트 범위 밖에서 일어난다. 
> - init(), doFilter(), destroy()
> - DelegatingFilterProxy의 등장으로 인해 스프링 빈으로 등록 가능
> > - [[Spring] 필터(Filter)가 스프링 빈 등록과 주입이 가능한 이유(DelegatingFilterProxy의 등장) - (2) _ 망나니개발자](https://mangkyu.tistory.com/221)
> - 공통된 보안, 인증, 인가 작업/ 모든 요청에 대한 로깅, 감사/ 이미지,데이터 압축 및 문자열 인코딩/ 스프링과 분리되어야 하는 기능, 
> - Spring Security는 필터를 이용해 인증, 인가에 사용하고 Spring MVC에 종속적이지 않다는 것이 특징이다. 
- 인터셉터는 SpringMVC 스펙에 포함되어 있습니다. 디스패처 서블릿이 컨트롤러를 호출하기 전/후에 요청과 응답을 참조하거나 가공할 수 있는 기능을 제공합니다. 즉 인터셉터는 스프링 컨텍스트 범위 내에서 동작합니다. 
> - preHandle(), postHandle(), afterCompletion()
> - 세부적인 보안 및 인증, 인가 작업/ API 호출에 대한 로깅, 감사/ 컨트롤러로 넘겨주는 데이터의 가공

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

#### JPA Propagation 전파단계를 설명해주세요.
- JPA Propagation은 트랜잭션 동작 도중 다른 트랜잭션을 호출하는 상황에 선택할 수 있는 옵션입니다. 
- Default 값은 REQUIRED이며 부모 트랜잭션 안에서 실행되며 부모 트랜잭션이 없을 경우 새로운 트랜잭션을 생성합니다.
- 이외의 값 REQUIRES_NEW, SUPPORTS, MANDATORY, NOT_SUPPORT, NEVER, NESTED 

<BR>

#### N + 1 쿼리 문제는 무엇이고 이것이 발생하는 이유와 이를 해결하는 방법을 설명해주세요.
- 1번 쿼리를 날렸을 때 의도치 않게 N개의 쿼리가 추가적으로 실행되는 것이다.
- 원인

 <BR>

1 . JPQL이 SQL을 생성할 때는 글로벌 Fetch 전략을 참고하지 않고 그대로 SQL 문으로 번역이 된다 
> - EX) `select u from User u ;` <BR>

2 . 이후 글로벌 페치 전략을 확인하고 연관된 엔티티를 조회하기 때문이다. 

> - findById의 경우 jpa가 내부적으로 join문에 대한 쿼리를 만들어서 반환을 하기 때문에 n+1이 발생하지 않는다. (inner join)
> - findBy~의 쿼리메소드 같은 경우에도 data jpa 내부에서 jpql이 만들어져서 나가고 findAll의 경우도 jpql이 만들어져서 나간다.

- 즉시로딩은 1, 2번 후 하위 엔티티가 즉시 조회되는 N개의 쿼리가 처리된다. 
- 지연 로딩은 1, 2번 후 지연 로딩 전략을 사용한 하위 엔티티를 프록시 처리해놓는다. 프록시인 하위 엔티티에 접근할 때 (unproxy하여 실제 객체에 접근한다) 해당 엔티티를 조회하기 위한 추가적인 N 쿼리가 발생한다.

- 해결방법
> - JPQL의 JOIN FETCH, 

```java
@Query("select o from Owner o join fetch o.cats")
List<Owner> findAllJoinFetch();
```

- @EntityGraph

```java
@EntityGraph(attributePaths = "cats")
@Query("select o from Owner o")
List<Owner> findAllEntityGraph();
```


- 그 외에 @Fetch(FetchMode.SUBSELECT), @BatchSize 조절 , QueryBuilder를 사용하는 방법이 있다. 


```java
// QueryDSL로 구현한 예제
return from(owner).leftJoin(owner.cats, cat)
                   .fetchJoin()
```

- [N+1 문제_Incheol's TECH BLOG](https://incheol-jung.gitbook.io/docs/q-and-a/spring/n+1)
- [JPA-모든-N1-발생-케이스과-해결책 _ @jinyoungchoi95](https://velog.io/@jinyoungchoi95/JPA-%EB%AA%A8%EB%93%A0-N1-%EB%B0%9C%EC%83%9D-%EC%BC%80%EC%9D%B4%EC%8A%A4%EA%B3%BC-%ED%95%B4%EA%B2%B0%EC%B1%85)

<br>

#### Spring에서 CORS 에러를 해결하기 위한 방법을 설명해주세요.
- Spring Filter를 사용하여 커스텀한 CORS 설정 방법,  WebMvcConfiguer를 구현한 Configuration 클래스를 만들어 addCorsMappings()을 재정의 방법, Spring Security에서 CorsConfigurationSource를 Bean으로 등록하여 config에 추가하여 해결하는 방법이 있다. 

<br>

#### Bean/Component 어노테이션에 대해서 설명해주시고, 둘의 차이점에 대해 설명해주세요.
- 두 어노테이션 모두 IoC 컨테이너에 Bean을 등록하기 위해 사용합니다.
- @Component : 개발자가 작성한 class를 기반으로 실행시점에 1회 생성합니다.(싱글톤)
> - @Controller, @Service, @Repository는 모두 @Component이며 자동으로 의존성을 주입합니다. 
- @Bean : 개발자가 작성한 method를 기반으로 메서등에서 반환하는 객체를 인스턴스 객체로 1회 생성합니다.(싱글톤)

<br>

#### POJO란 무엇인가요? Spring Framework에서 POJO는 무엇이 될 수 있을까요?
- POJO(Plain Old Java Object)는 프레임워크 인터페이스, 클래스를 구현하거나 확장하지 않은 단순한 클래스로 Java에서 제공하는 API 외에 종속되지 않습니다.
- 특정 환경에 종속되지 않아 코드가 간결하고 테스트 자동화에 유리합니다.
- 스프링에서는 도메인, 비즈니스 로직을 수행하는 대상이 POJO가 될 수 있습니다. 

<br>

#### Filter는 Servlet의 스펙이고, Interceptor는 Spring MVC의 스펙입니다. Spring Application에서 Filter와 Interceptor를 통해 예외를 처리할 경우 어떻게 해야 할까요?
- Filter는 DispatcherServlet 전달되기 전에 실행되기 때문에 Web Application에서 처리해야 합니다. 
- Interceptor는 Spring Context 내부에서 동작하기 때문에 @ControllerAdvice에서 @ExceptionHandler를 적용해서 처리할 수 있습니다. 

<br>

#### 의존성과 설정값을 생성자 인자로 주입해야 하는 이유에 대해 설명해주세요.
- 모든 의존성을 생성자를 통해 주입하면, 인스턴스 생성 시 즉시 어떠한 동작을 실행할 수 있습니다. 추가적인 설정이 필요하지 않으며 의존성, 설정값을 빠뜨리는 일이 발생하지 않고 테스트에 용이합니다. 
> - 스프링 컨테이너의 도움없이 테스트 코드 작성이 용이해진다. 만약 필드 주입, 수정자 주입으로 빈이 주입되어 있다면 Mockito를 이용해 목킹한 후 테스트를 진행해야 하지만  생성자 주입은 단순히 원하는 객체를 new 연산으로 생성하고 생성자에 넣어주면 됩니다. 


<br>

#### 더 학습할 내용


Spring Application을 구동할 때 메서드를 실행시키는 방법에 대해 설명해주세요.


Reference:
- [Backend-Interview-Question_ ksundong](https://github.com/ksundong/backend-interview-question)
- [ 생성자 주입 선택 시 테스트 코드 작성의 장점?_인프런](https://www.inflearn.com/questions/515588)
