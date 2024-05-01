---
layout: post
title: '[JAVA, SPRING] Spring Interceptor로 자동화 공격 막기'
subtitle: 'Spring, AU, 웹취약점'
date: 2024-04-15 14:33:00 +0900
categories: 'SPRING'
background: '/img/posts/etc/spring.jpg'
---

# 자동화 공격 
- 웹 어플리케이션을 대상으로 프로그램, 스크립트를 이용해 자동으로 반복하며 공격하는 방식이다.
- 특정 API가 무제한적으로 호출되면 서버가 정상적인 기능을 하지 못할 수 있으며, 과부화를 일으키게 된다.
- EX) 게시글 등록: 반복적으로 게시글을 등록하기 / SNS 발송 페이지: 계속 문자를 보내기 
- 나의 상황은 SMS 발송 페이지를 통해 문자를 무제한적으로 전송할 수 있었다.

<br>

# 고민

![interceptor](/img/posts/spring/interceptor.png)

- 상태정보를 유지할 수 있는 세션을 이용하기로 했고, 특정 API 호출을 가로챌 수 있는 Spring Interceptor를 이용하기로 했다.
- 서블릿 스펙의 필터는 디스패처 서블릿에 전달되기 전, 후에 작동하기 때문에 사용 포인트가 너무 앞쪽에 있게 될 것이라고 판단되었다. 또한 이미 인증, 인가를 위해 사용되는 필터가 존재하여 지나치게 성능에 영향을 주지 않을까? 생각하였다. 디스패처 서블릿에서 컨트롤러를 호출하는 사이에서 작동하는 것이 최적의 타이밍이라 생각했다. 
- 세션값 삭제? 재할당? 세션의 상태 유지는 기본적으로 클라이언트의 쿠키를 이용한다. 재할당을 매번해서 세션에 값을 저장할 필요가 없다고 판단하여서 매번 삭제하는 식으로 구현하여 1회성 작업으로 처리하였다.

<br>

# 순서 및 구현

- sms 전송창이 열리면 Interceptor에서 세션 토큰을 하나 생성, 상태 정보를 유지한다.
- 관련 정보(통신사, 휴대폰 번호)를 받고 sms 문자를 보내는 버튼을 클릭하면 다른 Interceptor에서 해당 세션의 값을 확인한다.
- 일치하면 문자전송, 아니면 화면 리프레쉬, 방금 사용한 토큰은 무조건 삭제한다. 

```java
// 토큰 생성 인터셉터
public class GeneratorInterceptor extends HandlerInterceptorAdapter {
    @Override
    boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        final String TOKEN_KEY = "token_key";
        final HttpSession session = request.getSession();
        String tokenValue = RandomStringUtils.randomAlphabetic(40);  //랜덤한 토큰값을 생성
        if(session.getAttribute(TOKEN_KEY) == null) {
            session.setAttribute(TOKEN_KEY, tokenValue);
        }
        return true;
    }
}
```

<br>

- 해당 세션값을 전달하기 위해서는 폼데이터와 헤더에 담는 방식이 있다. 
- 제이쿼리를 이용하여 세션값에 접근한다. 

##### 폼데이터

- 전송하는 폼데이터에 아래 코드를 넣어준다. 

```html
<input type="hidden" id="token_key" name="token_key" value="${sessionScope.token_key}"/>
```

##### 헤더
- 제이쿼리의 Ajax를 이용해 헤더에 키값을 붙여준다. 
> - [헤더에 담는 코드는 여기 참고!](https://lts0606.tistory.com/549)

<br>

```java
public class CheckInterceptor extends HandlerInterceptorAdapter {
    @Override
    boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        final String TOKEN_KEY = "token_key";
        final String REQUEST_URI = "/sms/sendSMS";
        final HttpSession session = request.getSession();
        
        final String REQUEST_URL = request.getRequestURL().toString(); 
        
        // URI 확인
        if(REQUEST_URL.equals(REQUEST_URI)) { 
            // 헤더라면 아래 처럼 접근
            // String csrf_key = request.getHeader("TOKEN_KEY");  //해더에 담긴 키

            // 폼데이터
            final String requestValue = request.getParameter(TOKEN_KEY); // name 값으로 전달
            
            if(!requestValue.equals(session.getAttribute(TOKEN_KEY).toString())){ 
                session.removeAttribute(TOKEN_KEY);
                response.sendError(999); // 특정 상태코드를 보내 이후 리프레시 처리 
                return false;  
            }
            // 성공해도 일회성 보장을 위해 삭제
            session.removeAttribute(TOKEN_KEY);        
            return true;
        }
        
        session.removeAttribute(TOKEN_KEY);
        response.sendError(999); // 특정 상태코드를 보내 이후 리프레시 처리 
        return false;  
    }
}
```

<br>

- 디스패처 서블릿 설정, dispatcher-servlet.xml

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc: mapping path="/sms/openSMSView"/>
        <bean class="com...interceptor.GeneratorInterceptor"/>
    </mvc:interceptor>

    <mvc:interceptor>
        <mvc: mapping path="/sms/sendSMS"/>
        <bean class="com...interceptor.checkInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

- [[Spring Interceptor] 커스텀 어노테이션과 Intercepter 구현](https://twer.tistory.com/entry/Spring-Interceptor-%EC%BB%A4%EC%8A%A4%ED%85%80-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98%EA%B3%BC-Intercepter-%EA%B5%AC%ED%98%84)

- 스프링3 버전 이후부터는 클래스 파일을 통해 설정을 지원한다. 위 링크 참고!

<br>

# 마무리
- 웹 취약점을 고려하는 방어적인 코드를 작성해야겠다. 
- 해당 기능을 통해 문자를 한번 성공적으로 보내고 다음에 반복적으로 요청하면 인터셉터를 통해 해당 요청을 걸러지게 되었다. 
- 추후에 횟수 5번의 기회를 주는 방식을 제공하는 것도 좋을 것 같다. 
  
<br>

Reference:

- [19. 자동화 공격(AU)](https://lts0606.tistory.com/549)
- [[Spring Interceptor] 커스텀 어노테이션과 Intercepter 구현](https://twer.tistory.com/entry/Spring-Interceptor-%EC%BB%A4%EC%8A%A4%ED%85%80-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98%EA%B3%BC-Intercepter-%EA%B5%AC%ED%98%84)