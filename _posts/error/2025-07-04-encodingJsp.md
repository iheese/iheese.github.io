---
layout: post
title: '[ERROR, SPING] 컨트롤러에서 한글 인코딩 시 확인해야 할 점'
subtitle: 'Controller, Encoding, 인코딩, 한글, JSP, JOSN, MappingJackson2HttpMessageConverter'
date: 2025-07-04 12:00:00 +0900
categories: [error, spring]
background: '/img/posts/etc/error.jpg'
---

## 한글 인코딩 문제

- 컨트롤러에서 ModelAndView 객체를 생성해서 JSP 화면으로 Model 데이터를 전달하려 했다. 근데 숫자나 영어일 때는 괜찮은데 자꾸 한글이 깨져서 전달되고 있었다.

<br>

```java
@Controller
public class TestController {
    @GetMapping("/test")
    public ModelAndView test() {
        ModelAndView mv = new ModelAndView();
        mv.addObject("test", "테스트");
        return mv;
    }
}
```

### 과정

1. 한글 데이터 Model 객체에 저장 
2. JSP에서 EL 표현식 등으로 렌더링 
3. HTML 응답 본문에 포함 
4. 클라이언트 브라우저가 렌더링

<br>

## 올바르게 UTF-8 한글 JSP에서 뿌려주기

- 아래 적어논 것들을 순서대로 잘 확인해보자.

#### 1. JSP 파일 인코딩 설정

- 해당 JSP 화면 위에 아래 코드 확인하기

```html
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
```

<br>

#### 2. spring 설정 파일 확인

- /WEB-INF/web.xml의 경우

```html
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
```

- Java config 클래스 파일인 경우

```java
@Bean
public FilterRegistrationBean<CharacterEncodingFilter> characterEncodingFilter() {
    CharacterEncodingFilter filter = new CharacterEncodingFilter();
    filter.setEncoding("UTF-8");
    filter.setForceEncoding(true);

    FilterRegistrationBean<CharacterEncodingFilter> registration = new FilterRegistrationBean<>(filter);
    registration.addUrlPatterns("/*");
    return registration;
}
```

<br>

#### 3. 서버 설정

- 구동하는 서버 `server.xml` 파일에 들어가서 확인하기, 보통 톰캣 서버에 있는 파일일 경우가 많다.  

```html
<Connector URIEncoding="UTF-8" ... />
```

- Connector 태그에 저 옵션이 있는지 확인하기
> - 톰캣을 새로 다운 받아 사용했던 나와 같은 경우 이 문제 때문이었다. 

<br>

## 추가) @ResponseBody 또는 @RestController을 이용해 JSON 응답으로 보낼 때

### 과정

1. 한글 데이터 객체에 담아 전달 
2. JSON 변환 (Jackson 등) 
3. UTF-8 바이트로 인코딩 
4. HTTP 응답 바디에 포함 
5. 브라우저 또는 JS가 파싱

```java
@RestController
public class TestController {
    @GetMapping("/test")
    public Map<String, String> test() {
        return Map.of("test", "테스트");
    }
}
```

1. @ResponseBody 또는 @RestController(각 메소드에 @ResponseBody가 붙음)를 사용하면 Spring은 반환 객체의 타입에 따라 HttpMessageConverter 선택한다.
2. 기본적으로 JSON 변환을 위해 MappingJackson2HttpMessageConverter 사용한다.
3. MappingJackson2HttpMessageConverter는 Converter는 application/json;charset=UTF-8 을 자동으로 Content-Type으로 설정하고, 본문을 UTF-8로 인코딩한다.

<br>

### converter 인코딩 설정

- 스프링 4 이후는 기본적으로 UTF-8 이지만 혹시 모를 설정 문제가 있는 버전이면 아래처럼 Bean 등록을 해줘야 한다.

```java
@Bean
public HttpMessageConverter<?> jsonConverter() {
    MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
    converter.setDefaultCharset(StandardCharsets.UTF_8);
    return converter;
}
```

<br>

Reference:

- [Sever.xml의 URIEncoding UTF-8 설정](https://highello.tistory.com/78)
