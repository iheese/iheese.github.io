---
layout: post
title: '[JAVA, SPRING WEBFLUX] SPRING WEBFLUX 란? '
subtitle: 'Spring MVC, WebFlux, WebClient'
date: 2023-09-27 15:00:00 +0900
categories: 'SPRING'
background: '/img/posts/etc/spring.jpg'
---

# Spring WebFlux
- Spring 5 부터 새롭게 추가된 모듈이며, 클라이언트/서버 구조에서 리액티브(reactive) 어플리케이션 개발을 위한 논블로킹 리액티브 스트림을 지원한다. 
> - 리액티브(reactive) 프로그래밍 : 데이터 스트림과 변경 사항 전파를 중심으로 하는 비동기 프로그래밍, 즉 데이터 흐름에 초점이 맞춰져 있고, 데이터를 비동기적으로 처리하며 이벤트 기반의 아키텍처로 실시간으로 데이터의 변화에 반응할 수 있는 프로그래밍이다. 
> - 논블로킹(non blocking) : A함수가 B함수를 호출해도 제어권은 그대로 자신이 가지고 있는다. 그래서 B 함수가 호출되어도 A함수는 계속 실행된다. 
- 적은 양의 쓰레드로 동시성을 처리하고, 보다 적은 하드웨어 자원으로 동시성을 처리하기 위한 위한 논블로킹(non-blocking) web 스택이 필요했기 때문에 만들어졌다.
- 함수형 프로그래밍 때문에 생기게 되었다. 자바 8 에서 함수형 API를 위한 람다식이 추가되었는데 이는 논블로킹 어플리케이션을 만들 때도 요긴하게 쓰인다. 비동기 로직의 선언적인 합성을 가능하게 하면서 연속적인 전달 스타일의 API를 작성할 수 있게 되었다.  

<br>

## Spring WebFlux VS Spring MVC

![spring-mvc-and-webflux-venn](https://github.com/iheese/iheese.github.io/assets/88040158/afeb2ae9-287a-46e8-bfe9-2f56de5764c8)

#### 공통점 
- @Controller
- Reactive 클라이언트
- Tomcat, Jetty, Undertow 같은 서버에서 실행할 수 있다.

#### 차이점 
- Spring MVC :
> - 명령형 논리(명령을 순서대로 처리), JDBC, JPA
> - 하나의 요청에 대해 하나의 쓰레드가 사용된다. 다량의 요청에 대비 쓰레드 풀을 생성해놓고 각 요청마다 쓰레드를 할당하여 처리한다.
> - 1 request : 1 thread, sync + blocking
- Spring WebFlux :
> - 기능적 엔드 포인트, 이벤트 루프, 동시성 모델
> - 논블로킹과 고정된 쓰레드 수로 모든 요청을 처리한다. 서버는 쓰레드 한 개로 운영하며, 디폴트로 CPU 코어 수 갯수의 쓰레드를 가진 워커 풀을 생성하여 워커 풀 내 쓰레드로 요청을 처리한다. 
> - many request : 1 thread, async + non-blocking

<br>

## R2DBC(Reactive Relational Database Connectivity)
- 관계형 DB에서 리액티브 프로그래밍을 가능하게 해주는 DB API
- R2DBC로 현재 사용할 수 있는 RDB
> - H2 (io.r2dbc:r2dbc-h2)
> - MariaDB (org.mariadb:r2dbc-mariadb)
> - Microsoft SQL Server (io.r2dbc:r2dbc-mssql)
> - MySQL (io.asyncer:r2dbc-mysql)
> - jasync-sql MySQL (com.github.jasync-sql:jasync-r2dbc-mysql)
> - Postgres (io.r2dbc:r2dbc-postgresql)
> - Oracle (com.oracle.database.r2dbc:oracle-r2dbc)

<br>

## Mono 와 Flux 
- Spirng WebFlux에서 사용하는 reactive library가 Reactor이고 Reactor Streams의 구현체이다. 
> - [리액티브 스트림즈는 비동기 스트림 처리를 위한 표준](http://www.reactive-streams.org/)
- Mono : 0 ~ 1개의 데이터 전달
- Flux : 0 ~ N개의 데이터 전달
> - 두 개 다 Reactor Streams의 publisher 인터페이스의 구현체이다. 

<br>

## Spring WebFlux 의 2가지 패턴

```gradle
implementation 'org.springframework.boot:spring-boot-starter-webflux'
```

- 의존성 추가


### @Controller 패턴

```java
@RestController
public class FluxController {

    @GetMapping("/")
    Flux<String> justFlux() {
        return Flux.just("Hello", "Flux");
    }
}
```

- `curl localhost:8080` 요청 시, text/plain 으로 응답 : HelloFlux
- `curl localhost:8080 -H 'Accept: text/event-stream'` Server-Sent Event 요청 시 : 아래와 같이 응답

```shell
data:Hello

data:Flux
```

- `curl localhost:8080 -H 'Accept: application/stream+json'` JSON Stream으로 요청 시 : HelloFlux
> - -i는 --include 응답 헤더를 출력

<br>

```java
@RestController
public class FluxController {
    
    @GetMapping("/stream")
    Flux<Map<String, Integer>> stream() {
        Stream<Integer> stream = Stream.iterate(0, i -> i + 1); 
        return Flux.fromStream(stream.limit(5)) 
                .map(i -> Collections.singletonMap("value", i));
    }
}
```

- `curl localhost:8080/stream` 일반 JSON 요청 시 : `[{"value":0},{"value":1},{"value":2},{"value":3},{"value":4}]`
- `curl localhost:8080/stream 'Accept: text/event-stream'` Server-Sent Event 요청 시 : 아래와 같이 응답

```shell
data:{"value":0}

data:{"value":1}

data:{"value":2}

data:{"value":3}

data:{"value":4}
```

- `curl localhost:8080/stream -H 'Accept: application/stream+json'` JSON Stream 요청 시 : 아래와 같이 응답

```shell
{"value":0}
{"value":1}
{"value":2}
{"value":3}
{"value":4}
```

- @Controller 패턴은 Spring MVC 에서 자주 사용했던 방식과 똑같이 처리된다. 
- 다만 리턴 객체가 Mono, Flux 이며 데이터 스트림 형태로 처리되는 차이점만 존재한다. 

<br>

### Router Functions
- 경로와 핸들러 함수(람다)의 조합으로 라우팅을 정의한다.
- 같은 요청을 컨트롤러 패턴과 라우터 패턴 모두 쓰이면 Spring Boot 2.7.16 (자바 8) 기준  라우터가 우선 순위를 가지고, Spring Boot 3.1.2 (자바 17) 기준 컨트롤러가 우선 순위를 가졌다.
> - 두 패턴은 공존할 수 없기 때문에 같은 요청으로 구현한 것은 무시되었다. 

```java
@Bean
RouterFunction<ServerResponse> routes() {
    return RouterFunctions.route(RequestPredicates.GET("/"), req -> ServerResponse
            .ok().body(Flux.just("Hello", "Flux"), String.class));
}
```

- @Bean 으로 메소드에 라우터 패턴 적용하기
- `curl -i localhost:8080` 요청 시, text/plain 으로 응답 : HelloFlux

```java

@Component
public class FluxHandler {

    @Bean
    public RouterFunction<ServerResponse> routes() {
        return route(GET("/"), this::hello)
        .andRoute(GET("/stream"), this::stream)
        .andRoute(POST("/upper"), this::upper);
    }

    public Mono<ServerResponse> hello(ServerRequest req) {
        return ok().body(Flux.just("Hello", "Flux"), String.class);
    }

    public Mono<ServerResponse> stream(ServerRequest req) {
        Stream<Integer> stream = Stream.iterate(0, i -> i + 1);
        Flux<Map<String, Integer>> flux = Flux.fromStream(stream.limit(5))
                .map(i -> Collections.singletonMap("value", i));
        return ok().contentType(MediaType.APPLICATION_NDJSON)
                .body(fromPublisher(flux, new ParameterizedTypeReference<Map<String, Integer>>(){}));
    }

    public Mono<ServerResponse> upper(ServerRequest req) {
        Mono<String> body = req.bodyToMono(String.class).map(String::toUpperCase);
        return ok().body(body, String.class);
    }
}
```

- @Component 로 클래스에 라우터 패턴 적용하기
- `curl localhost:8080/stream` 요청 시 : 아래와 같이 응답

```java
{"value":0}
{"value":1}
{"value":2}
{"value":3}
{"value":4}
```

- `curl localhost:8080/upper -d hello` 요청 시 : HELLO

<br>

### RouterConfig 로 나누어서 처리하기

![webflux](https://github.com/iheese/iheese.github.io/assets/88040158/0aee62ee-5b4c-4a9a-9947-16bee2d68489)

- 예시

```java
@Configuration
public class RouterConfig {

    @Bean
    RouterFunction<ServerResponse> routes(UserHandler handler) {
        return route(GET("/handler/users").and(accept(MediaType.APPLICATION_JSON)), handler::getAllUsers)
                .andRoute(GET("/handler/users/{userId}").and(contentType(MediaType.APPLICATION_JSON)), handler::getUserById)
                .andRoute(POST("/handler/users").and(accept(MediaType.APPLICATION_JSON)), handler::create)
                .andRoute(PUT("/handler/users/{userId}").and(contentType(MediaType.APPLICATION_JSON)), handler::updateUserById)
                .andRoute(DELETE("/handler/users/{userId}").and(accept(MediaType.APPLICATION_JSON)), handler::deleteUserById);
    }
}
```

- 실제 로직을 진행하는 핸들러 객체를 받아, 컨트롤러에서 요청을 받고 응답하는 일을 처리하듯 라우터에 연결된 핸들러 메소드를 사용한다. 

```java
@Component
@RequiredArgsConstructor
public class UserHandler {

    private final UserService userService;

    public Mono<ServerResponse> getAllUsers(ServerRequest request) {
        return ServerResponse
                .ok()
                .contentType(MediaType.APPLICATION_JSON)
                .body(userService.getAllUsers(), User.class);
    }

    public Mono<ServerResponse> getUserById(ServerRequest request) {
        return userService
                .findById(request.pathVariable("userId"))
                .flatMap(user -> ServerResponse
                        .ok()
                        .contentType(MediaType.APPLICATION_JSON)
                        .body(user, User.class)
                )
                .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> create(ServerRequest request) {
        Mono<User> user = request.bodyToMono(User.class);

        return user
                .flatMap(u -> ServerResponse
                        .status(HttpStatus.CREATED)
                        .contentType(MediaType.APPLICATION_JSON)
                        .body(userService.createUser(u), User.class)
                );
    }

    public Mono<ServerResponse> updateUserById(ServerRequest request) {
        String id = request.pathVariable("userId");
        Mono<User> updatedUser = request.bodyToMono(User.class);

        return updatedUser
                .flatMap(u -> ServerResponse
                        .ok()
                        .contentType(MediaType.APPLICATION_JSON)
                        .body(userService.updateUser(id, u), User.class)
                );
    }

    public Mono<ServerResponse> deleteUserById(ServerRequest request){
        return userService.deleteUser(request.pathVariable("userId"))
                .flatMap(u -> ServerResponse.ok().body(u, User.class))
                .switchIfEmpty(ServerResponse.notFound().build());
    }
}
```

> - 위는 reflectoring 의 예시를 가져왔다. 아래 주소에서 원본글을 확인할 수 있다. 

<br>

## WebClient
- Spring WebFlux에서 HTTP Client 로 사용되는 비동기적으로 작동하는 모듈
> -  HTTP Client 라이브러리를 상속받고 디폴트로 Netty의  HTTP Client를 사용한다.
- 동기적인 작업은 RestTemplate, 비동기적인 작업은 WebClient 가 적합하다. 

- 예시

```java
@Component
@RequiredArgsConstructor
public class WebClientUtil {

    private final WebClient webClient;

    public WebClient.ResponseSpec getFakeUsers() {
        return webClient
                .get()
                .uri("https://randomuser.me/api/")
                .retrieve();
    }

    public Mono<User> postUser(User user) {
        return webClient
                .post()
                .uri("http://localhost:9000/api/users")
                .header("Authorization", "Basic MY_PASSWORD")
                .accept(MediaType.APPLICATION_JSON)
                .body(Mono.just(user), User.class)
                .retrieve()
                .bodyToMono(User.class)
                .log()
                .retryWhen(Retry.backoff(10, Duration.ofSeconds(2)))
                .onErrorReturn(new User())
                .doOnError(throwable -> log.error("Result error out for POST user", throwable))
                .doFinally(signalType -> log.info("Result Completed for POST User: {}", signalType));
    }
}
```

- 메소드 방식, uri로 요청하여 원하는 값을 비동기적으로 처리할 수 있을 뿐만 아니라 에러 처리까지 가능하다.
- [WebClient 사용에 관한 방법과 메소드 설명](https://adjh54.tistory.com/233)

<br>

Reference:

- [WebFlux란 무엇인가? _ 코딩무니](https://devmoony.tistory.com/174)
- [Spring WebFlux의 간단한 사용법](https://www.devkuma.com/docs/spring-webflux/)
- [Getting Started with Spring WebFlux](https://reflectoring.io/getting-started-with-spring-webflux/)
- [A functional endpoint in Spring WebFlux](https://jstobigdata.com/spring/a-functional-endpoint-in-spring-webflux/)
- [WebClient 예시 코드](https://github.com/thombergs/code-examples/blob/master/spring-webflux/src/main/java/io/reflectoring/springwebflux/client/WebClientUtil.java)