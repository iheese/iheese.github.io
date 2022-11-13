---
layout: post
title: '[JAVA, SPRINGBOOT] SPRING SECURITY 훑어보고 커스텀하기'
subtitle: 'Authentication, Authorization, SecurityFilterChain, UserDetailsService, UserDetails'
date: 2022-07-27 12:00:00 +0900
categories: 'SPRING'
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

- 의존성만 추가해도 Spring Security 기본 설정들이 적용되며, 어플리케이션 실행시 user라는 기본 계정이 생성되고 Spring Security가 제공하는 기본 로그인 화면이 나오게 된다.
- 기본 아이디는 `user`이며, 비밀번호는 콘솔창에 Using generated security password 에 나오게 된다.

<br>

### Spring Security 순서

![springsecurity](/img/posts/spring/springsecurity.png){: width="800"}

1, 2 . 로그인 요청이 들어오면 AuthenticationFilter에서 입력된 아이디와 비밀번호는 UsernamePasswordToken으로 바꾸어 전달된다. 

3 . AuthenticationManager(구현체는 ProviderManager)는 UsernamePasswordToken 인증을 처리할 AuthenticationProvider를 가지고 있다. (여러 개 존재할 수 있다.)

4 . AuthenticationManager는 토큰을 전달하여 AuthenticationProvider의 구현체들을 통해 인증을 요구한다.

5 . AuthenticationProvider는 UserDetailsService를 통해 사용자 정보를 조회한다.

6, 7 . UserDetailsService은 UserDetails를 리턴해주는 하나의 메소드(loadUserByUsername)를 구현해야 한다. 일반적으로 UserRepository를 주입받아 UserDetails를 리턴한다.

- UserDetails는 인가 권한이 추가된 User 객체이다.
> - UserDetails는 인터페이스고 먼저 만든 UserVO 객체가 있다면 주입해줘도 되고, 인터페이스 자체를 구현해도 된다.

8 . AuthenticationProvider가 인증에 성공하면 성공한 UsernameAuthenticationToken을 생성하여 AuthenticaionManager에게 전달한다.

9 . 그리고 AuthenticaionManager는 AuthenticationFilter에게 토큰을 전달한다.

10 . 토큰은 LoginSuccessHandler로 전달되고 SecurityContextHolder에 저장된다. 
- SecurityContextHolder은 인증한 내용을 가지고 있다.
> - SecurityContextHolder은 SecurityContext를 thread와 연결해주고 있다.
> - SecurityContext은 Authentication에 대한 get/set() 메소드가 정의되어 있다. (Authentication : user의 인증 정보)

<br>

### Spring Security 커스텀하기

- Spring Security는 위 순서처럼 복잡하지만 필요한 기능만을 이용해 사용할 수 있다.
- 실습은 간단하게 SecurityFilterChain을 이용한 설정파일 - UserDetailsService(Service 단) - UserDetails(객체)로 진행됩니다.
> - [Spring Security 처리 과정 및 구현 예제_
MangKyu's Diary](https://mangkyu.tistory.com/77) 을 참고하면 구현 예제에 대한 설명을 자세히 볼 수 있다. (좋은 자료 감사합니다.)
> - [spring security guide](https://spring.io/guides/gs/securing-web/) 를 참고하여 간단하게 실습해봐도 좋을 것 같다.
- SecurityFilterChain : Spring Security를 커스텀할 설정 파일이다.
> - Bean 등록을 통해 커스텀할 수 있다. 
- UserDetailsService : 인터페이스, Repository로부터 객체의 아이디를 이용해 찾아 UserDetails 객체를 리턴해준다.
- UserDetails : 인터페이스, 일반 객체에 인가 권한이 추가된 형태이다. 
> - 일반 객체를 생성자 주입하여 사용하거나 인터페이스 자체를 구현해준 뒤 사용하면 된다. 

<br>
- Spring Security 커스텀을 위해서는 환경설정 파일을 통해 설정을 재정의해줘야 한다.

WebSecurityConfig.java

```java
@EnableWebSecurity //Spring Security 커스텀에 필수적인 Annotation
public class WebMvcConfiguration { 
	
	@Bean
	protected SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		
		http.authorizeRequests()
		.antMatchers("/", "/auth/**", "/js/**", "/image/**", "/webjars/**")
		.permitAll()
		// 위에서 언급한 경로 외에는 모두 인증을 거치도록 설정
		
        .and()	
		.authorizeRequests().anyRequest().authenticated()
	
		// 시큐리티가 제공하는 기본 로그인 화면은 CSRF 토큰을 무조건 전달
		// 하지만 사용자 정의 로그인 화면에서는 CSRF 토큰을 전달하지 않게 설정
		.and()
		.csrf().disable()
	
		// 사용자가 만든 로그인 화면 이용
		.formLogin().loginPage("/auth/login")
        
		//로그인 실패 핸들러 
		.failureHandler(authenticationFailureHandler)
	
		// 로그아웃 설정
		.and()
		.logout().logoutUrl("/auth/logout").logoutSuccessUrl("/");

		return http.build();
			
	}
}
```

- WebSecurityConfigurerAdapter를 사용했었지만 현재 Deprecated 되어 
Spring Security 5.7 이후부터 SecurityFilterChain를 Bean 등록하여 사용하게 된다. 
- 참고 자료 : [WebSecurityConfigurerAdapter에서 SecurityFilterChain로_ 공식문서](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)
- 위 설정 파일에 로그인 성공 핸들러`LoginSuccessHandler`, 로그인 실패 핸들러인 `AuthenticationFailureHandler`, 비밀번호 인코드 `BCryptPasswordEncoder` 등의 여러 기능을 Bean 추가 혹은 클래스를 주입 받아 사용할 수 있다.

<br>

UserDetailsServiceImpl.java

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {
	
	@Autowired
	private UserRepository userRepository;
	
	//UserDetails 객체 리턴 (인가가 추가된 객체)
	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		User principal = userRepository.findByUsername(username).get();
		return new UserDetailsImpl(principal);
	}
}
```

- 인가가 포함된 유저 객체를 불러오는 Service 클래스이다.

<br>

UserDetailsImpl.java

```java
@Getter
@Setter
public class UserDetailsImpl implements UserDetails {
	private static final long serialVersionUID = 1L;
	
	// User 엔티티 타입의 참조변수 선언
	private User user;
	
	//UserDetails 객체 생성
	public UserDetailsImpl(User user) {
		this.user = user;
	}
	
	// User 엔티티가 가지고 있는 권한 목록을 저장하여 리턴
	@Override
	public Collection<? extends GrantedAuthority> getAuthorities() {
		// 권한 목록을 저장할 컬렉션 
		Collection<GrantedAuthority> roleList = new ArrayList<>();
		
		// 권한 설정
		roleList.add(new GrantedAuthority() {
			private static final long serialVersionUID = 1L;

			@Override
			public String getAuthority() {
				return "ROLE_" + user.getRole();
			}
		});
		return roleList;
	}

	@Override
	public String getPassword() {
		// {noop}은 비밀번호를 암호화하지 않도록 하는 접두사
		return "{noop}" + user.getPassword();
	}

	@Override
	public String getUsername() {
		return user.getUsername();
	}
	
	// 계정이 만료됬는지 여부를 리턴
	@Override
	public boolean isAccountNonExpired() {
		return true;
	}

	// 계정이 잠겨있는지 여부를 리턴
	@Override
	public boolean isAccountNonLocked() {
		return true;
	}

	// 비밀번호가 만료됬는지 여부를 리턴
	@Override
	public boolean isCredentialsNonExpired() {
		return true;
	}

	// 계정의 활성화 여부를 리턴
	@Override
	public boolean isEnabled() {
		return true;
	}
}

```

- UserDetails 인터페이스를 구현하여 만들어도 되고 기존의 User 객체가 있다면 생성자 주입을 이용해도 된다. 

<hr>

- 위 실습에서는 간단한 설정 파일과 서비스단, 인가 처리된 객체를 살펴보았다.
- 프로그램이 커지면서 사용자가 많아지면 발생할 수 있는 보안, 유저 관련 문제들에 대해 효율적으로 처리할 수 있을 것 같다.
- 인증, 인가, 보안과 관련된 필요한 기능을 모듈처럼 가져와서 사용할 수 있고, 개발자에게 체계적으로 많은 옵션을 제공하는 점이 큰 장점이라고 생각된다. 

<br>

Reference:
- [inflean_img](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC_renew)
- [[SpringBoot] Spring Security란?](https://mangkyu.tistory.com/76)
- [[SpringBoot] Spring Security 처리 과정 및 구현 예제}](https://mangkyu.tistory.com/77)
- [스프링 시큐리티 이것부터 고쳐 쓰세요_백기선](https://www.youtube.com/watch?v=fG21HKnYt6g)
- [Spring Security Custom 사용법 (Code-Based) _ lhm0812](https://m.blog.naver.com/lhm0812/221057255249)

