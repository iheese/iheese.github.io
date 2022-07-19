---
layout: post
title: '[JAVA, SPRINGBOOT] SPRING SECURITY 훑어보기'
subtitle: ''
date: 2022-07-15 12:00:00 +0900
categories: 'JAVA'
background: '/img/posts/etc/spring.jpg'
---

- 모든 실습은 Eclipse Spring Tool Suite 4 에서 진행되었습니다.

<br>

### Spring Security
- Spring 기반의 어플리케이션 보안(인증과 인가)을 담당하는 프레임워크이다.
> - 인증(Authentication) : 식별 가능한 정보로 등록된 유저의 신원을 입증하는 과정을 의미한다.
> - 인가(Authorization) : 인증된 사용자에 대한 자원 접근 권한을 확인한다.
> > - Ex) 많은 회사가 모여 있는 건물에 들어갈 때 사원증이 필요하다.
> > - 사원증을 이용해 회사 건물에 들어가는 것을 인증,
> > - 건물 내 많은 회사 중 나의 회사 층만 들어갈 수 있는 것은 인가이다. 

<br>

### Spring Security 사용하기

- Maven을 사용한다면 pom.xml에 아래와 같은 의존성을 추가해줘야 한다.

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-test</artifactId>
			<scope>test</scope>
		</dependency>
```

- Gradle을 사용한다면 build.gradle에 아래와 같은 의존성을 추가해줘야 한다.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.springframework.security:spring-security-test'
```

- 의존성만 추가해도 Spring Security 기본 설정들이 적용되며, 어플리케이션 실행시 user라는 객체가 생성되어 Spring Security가 제공하는 기본 로그인 화면이 나오게 된다.
- 기본 아이디는 `user`이며, 비밀번호는 콘솔창에 Using generated security password 에 나오게 된다.

<br>

### Spring Security 순서

![springsecurity](/img/posts/spring/springsecurity.png)

- Http 요청이 들어오면 AuthenticationFilter를 거쳐 UsernamePasswordToken을 발급해준다.
- AuthenticationManager는 UsernamePasswordToken 인증을 처리할 AuthenticationProvider 여러 개를 가지고 있다.
- AuthenticationProvider을 통해 UsernamePasswordToken의 아이디를 조회하고 UserDetailsService에 아이디를 기반으로 데이터를 조회한다.
> - UserDetails는 인터페이스고 일반 UserVO 객체에 인가 권한이 추가된 객체이다.
-  UserDetailsService은 데이터를 리턴해주고, AuthenticationProvider에서 조회한 데이터와 입력받은 비밀번호가 일치하는지 확인하고 일치하면 인증된 토큰을 생성하여 반환해준다.
-  AuthenticationFilter에게 인증된 토큰이 리턴되고 LoginSuccessHandler로 전달된다.
-  LoginSuccessHandler는 SecurityContextHolder에 저장하면 인증 과정이 끝난다.
> - 로그인 성공 후 세션을 이용해 로그인 반복을 줄인다. 

### Spring Security 커스텀하기

<br>

- Spring Security에 집중된 실습을 위해 Repository, Service 단은 고려하지 않았고 일반적으로 Controller - Service - Repository - DB 로 연결되어 있다고 이해해주시면 됩니다. 
> - 각자 JPA, MyBatis, JDBC 등을 이용하여 구현하면 됩니다.
> - [spring security guide](https://spring.io/guides/gs/securing-web/) 를 참고하여 실습해봐도 좋을 것 같습니다.


- 실습 큰 틀은 Controller(사용자 요청, 화면 이동 처리) + WebSecurityConfig(사용자 요청에 대한 security 처리, 화면 연결 설정 등등) + UserDetailsService로부터 AuthenticationManagerBuilder을 통해 인증 객체 연결 
> - AuthenticationManagerBuilder는 AuthenticationProviders를 쉽게 추가할 수 있도록 인증 메커니즘을 설정하는데 사용됩니다. 
-  UserDetailsService(인터페이스, Repository로부터 일반 객체를 받고 UserDetails 객체를 리턴해준다. )
- UserDetails 객체(인터페이스, 일반 객체를 받아 권한 설정을 더해준다.)
- User(일반 VO객체)


<br>

- 먼저 로그인, 회원가입 하는 Controller를 만들어본다.

AuthController.java

```java
@Controller
public class AuthController {

	@Autowired
	private UserService userService;
	
	// 로그인 화면으로 이동
	@GetMapping("/login") 
	public String login() {
		return "views/login";
	}
	
	// 회원가입 화면으로 이동
	@GetMapping("/insertUser") 
	public String insertUser() {
		return "views/insertUser";
	}
	
	// 회원 가입 처리
	@PostMapping("insertUser")
	public @ResponseBody String insertUser(@RequestBody User user) {
		// username으로 등록된 회원이 있나 검색한다. 
		User findUser = userService.getUser(user.getUsername());
		
		if(findUser.getUsername() == null) {
			user.setRole("USER");
			userService.insertUser(user);
			return user.getUsername() + " 회원 가입 성공";
		} else {
			return user.getUsername() + " 아이디는 이미 존재합니다";
		}
	}
```

<br>
- Spring Security 커스텀을 위해서는 환경설정 파일을 통해 설정을 재정의해줘야 한다.

WebSecurityConfig.java

```java
@Configuration //설정 클래스
@EnableWebSecurity //Spring Security 커스텀에 필수적인 Annotation
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	
    @Autowired
	private UserDetailsServiceImpl userDetailsService;
    
	
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth.userDetailsService(userDetailsService);
	}
    
    @Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.antMatchers("/", "/home").permitAll()
				.anyRequest().authenticated()
				.and()
			.formLogin()
				.loginPage("/login")
				.permitAll()
				.and()
			.logout()
				.permitAll();
	}
```


<br>
Reference:
- [inflean_img](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC_renew)
- [[SpringBoot] Spring Security란?](https://mangkyu.tistory.com/76)
- [[SpringBoot] Spring Security 처리 과정 및 구현 예제}](https://mangkyu.tistory.com/77)
- [스프링 시큐리티 이것부터 고쳐 쓰세요_백기선](https://www.youtube.com/watch?v=fG21HKnYt6g)
