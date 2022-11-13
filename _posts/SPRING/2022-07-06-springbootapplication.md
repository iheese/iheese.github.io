---
layout: post
title: '[JAVA, SPRINGBOOT] @SpringBootApplication, 외부 설정 파일'
subtitle: '@SpringConfiguration, @EnableAutoConfiguration, @ComponentScan, @Value, Environment, @ConfigurationProperties, .yml'
date: 2022-07-06 12:00:00 +0900
categories: 'SPRING'
background: '/img/posts/etc/spring.jpg'
---

- 모든 실습은 Eclipse Spring Tool Suite4 기준으로 진행되었습니다.

<br>

###  SpringBoot Project 시작하기

- New - Spring Starter Project를 통해 SpringBoot Project를 시작할 수 있다. 
- 빌드 타입(Maven, Gradle), Packing(Jar, War), Java version, Language 등을 설정할 수 있다.
- 빌드 프로그램을 통해 Project Dependencies를 추가할 수 있다.(필요한 라이브러리를 선택하면 된다.)
- 클래스 생성 후 클래스의 src/main/java/패키지/[className]Application.java 파일에서 오른쪽 클릭 - Run as - Spring Boot App을 통해 실행할 수 있다.

TestApplication.java

```java
package com.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class TestApplication {

	public static void main(String[] args) {
		SpringApplication.run(TestApplication.class, args);
	}
}

```

- Spring Boot 내부 Tomcat Server가 존재하고 스프링 기반의 웹 어플리케이션이 동작한다.
- 다른 설정없이 웹 어플리케이션이 실행될 수 있었던 것은 `@SpringBootApplication` 덕분이다. 

<br>

## @SpringBootApplication
- ctrl를 누른 상태에서 @SpringBootApplication을 누르면 해당 Annotation에 대한 코드를 확인할 수 있다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
	//아래 내용 생략...

```

- 중요한 요소는 @SpringBootConfiguration, @EnableAutoConfiguration, @ComponentScan 이다. 

<br>

##### @SpringBootConfiguration 
- SpringBoot에 관한 @Configuration이라고 생각하면 된다.
> - TestApplication.java이 설정 파일 클래스 역할도 포함하고 있다.
> - @SpringBootConfiguration은 자동적으로 설정들이 위치하는 것을 허락한다.
> - 나중에 유닛, 통합 테스트를 진행할 때 유용하게 적용될 수 있다.  

<hr>

- Spring Boot 어플리케이션이 운영되려면 2종류의 Bean이 필요하다.
> - 다른 종류의 객체를 위해 Container가 2번 생성된다.
- 첫번째는 사용자 정의 Bean(객체), 두번째는 스프링 프레임워크 제공 Bean(객체)이다.
- 순서는 @ComponentScan, @EnableAutoConfiguration 순으로 실행된다. 

#### @ComponentScan
- 사용자가 정의한 Bean을 초기화하고 메모리에 올려주는 Annotation이다. 
- 전에 .xml 파일에서 설정했던 Component-Scan과 같은 기능을 가진다.

<br>

#### @EnableAutoConfiguration
- 스프링 프레임워크가 제공하는 Bean을 초기화하고 메모리에 올려주는 Annotation이다. 
- 스프링 프레임워크가 제공하는 Bean들은 아래 Annotation의 조건으로 구성되어 있고 조건에 따라 생성된다.
> - 아래 Annotation을 통해 Bean 설정 파일의 조건을 재정의해줄 수 있고 사용자 정의 스타터를 만들 때 유용하게 사용될 수 있다. 
> > - @ConditionalOnWebApplication : 웹 어플레케이션 타입을 체크하여 설정 클래스에 적용한다.(SERVLET(Spring Boot App), NONE(일반 Java Application))
> > - ConditionalOnMissingBean : 특정 Bean이 메모리에 없을 때 현재 설정 클래스에 적용한다.
> > - @ConditionlOnBean : 해당 Bean이 포함되어 있을 때 설정 클래스에 적용한다.
> > - @ConditionalOnClass : 해당  클래스가 클래스 경로에 존재할 때 설정 클래스를 적용한다.
> > - @AutoConfigureOrder : 자동 설정 클래스의 우선순위를 지정한다.(Ordered.HIGHEST_PRECEDENCE + 5 : 가장 높은 우선순위보다 5단계 높게 설정)
- 사전 정의 파일 위치 : Dependencies > spring-boot-autoconfigure > META-INF > spring.factories 


<br>

### 외부 설정 파일(application.properties)
- 클래스 내 src/main/resources/application.properties에 위치한다.
- Java 코드보다 외부 설정 파일의 우선순위가 높다.
- Properties는 내부적으로 Setter Injection에 의해 반영된다.

application.properties 예시

```properties
# Application 모드 설정
spring.main.web-application-type=servlet 
#기본 Spring Boot App, none은 그냥 Java Application으로 실행된다.

#Banner Setting, 끌 수도 있고 내가 원하는 banner을 넣어줄 수도 있다.
spring.main.banner-mode=off
spring.banner.location=banner/banner.txt

#Bean Overriding 설정(중복 객체 덮어쓰기)
spring.main.allow-bean-definition-overriding=true

#View Resolver 설정
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.html

#Server Port(기본은 8080)
server.port=8080
```

- [참고! 배너 생성 사이트, 복붙하여 만들 수 있다.](https://patorjk.com/software/taag/#p=display&f=Graffiti&t=Type%20Something%20)

<br>

### 설정 파일에서 값 가져오기
#### @Value

application.properties

```properties
test.name=spring
test.rd=${random.int} # 랜덤으로 값을 줄 수도 있다. 
```

- java 파일 내에서 @Value로 접근할 수 있다. 

```java
@Component
public class Test1 implements ApplicationListener<ApplicationStartedEvent> {

    @Value("${test.name}")
    private String name;
    @Value("${test.rd}")
    private int randomNum;

    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println(name);
        System.out.println(randomNum);
    }
}
```

<br>

#### Environment

- Spring Boot가 실행되면서 설정 파일의 필요한 값을 ApplicationEnvironment에 담는다.(ApplicationEnvironment은 Environment를 상속하고 있다.)

application.properties

```properties
test.name=spring
```


```java
@Component
public class Test2 implements ApplicationListener<ApplicationStartedEvent>{
	
	@Autowired
	public Environment environment;
	
	@Override
	public void onApplicationEvent(ApplicationStartedEvent event) {
		System.out.println(environment.getProperty("test.name"));
	}
}
```

<br>

#### @ConfigurationProperties

application.properties

```properties
test.name=spring
```

```java
@Data //lombok
@Component
@ConfigurationProperties("test")
public class TestObject{
    private String name;
}
```

```java
@Component
public class test3 implements ApplicationListener<ApplicationStartedEvent>{
	
	 @Autowired
	 private TestObject testObject;
	
	@Override
	public void onApplicationEvent(ApplicationStartedEvent event) {
		System.out.println(testObject.getName());
	}
}


```

- 위 방법을 DataSource 적용할 때도 사용할 수 있다.

<br>

application.properties

```properties
#Data Source
data.source.jdbc.driverClassName=org.h2.Driver
data.source.jdbc.url=jdbc:h2:tcp://localhost/~/test
data.source.jdbc.username=username
data.source.jdbc.password=password
```

```java
@Data //lombok
@ConfigurationProperties(prefix="data.source")
//data.source로 시작하는 설정 값과 매핑
public class JDBCProperties {
	private String url;
	private String driverClassName;
	private String username;
	private String password;
}
```


```java
@Configurable
@EnableConfigurationProperties(JDBCProperties.class)
public class AutoConfiguration {
	
	@Autowired
	private JDBCProperties properties;
    
    //아래 내용 생략
```

<br>

### 여러 외부 설정 파일 적용하기

- @PropertySource을 이용하면 된다.

```java
@SpringBootApplication
@PropertySource(value = {"test1.properties", "test2.properties"})
public class PropertiesApplication {

    public static void main(String[] args) {
        SpringApplication.run(PropertiesApplication.class, args);
    }
}
```

<br>

### application.yml
- application.properties 대신 application.yml 이라는 파일 타입이 많은 곳에서 설정 파일 타입으로 사용되고 있다.
- 한 줄이 내려갈 때 띄어쓰기 X 2으로 구분하며, 반복되는 코드를 줄일 수 있다.

```yml
#Data Source
data:
  source:
    jdbc:
      driverClassName: org.h2.Driver
      url: jdbc:h2:tcp://localhost/~/test
      username: username
      password: password
      
#Server
server:
  port: 7777
  servlet: 
    context-path: /
    encoding:
      charset: UTF-8
```

<br>

Reference:
- [inflean_img](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC_renew)
- [Guide to @SpringBootConfiguration in Spring Boot_ Baedung](https://www.baeldung.com/springbootconfiguration-annotation)
- [[Spring Boot] 외부설정1 - Application.properties_ air.log](https://velog.io/@max9106/Spring-Boot-%EC%99%B8%EB%B6%80%EC%84%A4%EC%A0%95-uik69crax3)
- [[Springboot] 설정 파일에서 값 가져오는 방법_haerong22](https://velog.io/@haerong22/Springboot-%EC%84%A4%EC%A0%95-%ED%8C%8C%EC%9D%BC%EC%97%90%EC%84%9C-%EA%B0%92-%EA%B0%80%EC%A0%B8%EC%98%A4%EB%8A%94-%EB%B0%A9%EB%B2%95)