---
layout: post
title: "[JAVA] JSP와 SERVLET"
subtitle: "JSP, Servlet, redirect, forward"
date: 2022-02-08 12:00:00 +0900
categories: "JAVA"
---

### JSP(Java Server Pages)
- HTML 문서 안에 Java 코드를 넣어 동적 웹페이지를 설정하는 웹 어플리케이션 도구이다. 
- JSP는 서블릿 기술로 변환되어 사용하한다.

<BR>
#### JSP Lifecycle

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
hello
<%
	System.out.println("jspService()");
%>

<%!
public void jspInit() {
	System.out.println("jspInit()!");
}

public void jspDestroy() {
	System.out.println("jspDestroy()");
}
%>

</body>
</html>

```

- JSP 실행순서
> 1. 브라우저가 웹서버에 대한 JSP에 대한 요청 정보를 전달한다.
> 2. 브라우저가 요청한 JSP가 최초일 경우만 JSP로 작성한 코드를 서블릿 코드로 변환한다.(java 파일 생성)
> 3. 서블릿 코드를 컴파일한다. (class 파일 생성)
> 4. 클래스 로딩하고 인스턴스 생성한다.
> 5. 서블릿이 실행되어 요청을 처리하고 응답 정보를 생성한다. 

- servlet 라이프 사이클의 메소드
> - init(),  service(), destroy()
- JSP 라이프 사이클의 메소드 (jsp가 서블릿으로 변경된 파일에서 사용되는 메소드)
> - _jspinit(),  _jspservice(), _jspdestroy()
- 두 라이프 사이클 모두 생성, init()은 처음에만 발생하며 그 후 메모리에 로딩되어 요청시 객체를 재활용하고 종료 혹은 변경시 destroy()가 한 번 발생하게 된다. 

<br>

#### JSP 문법

- 선언문(DeClaration): <%! %> , 전역변수 선언 및 메소드 선언
- 스크립트릿(Scriptlet): <% %>, 프로그래밍 코드 기술
- 표현식(Experssion): <%= %>, 화면에 출력할 내용 기술

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<% 
for(int i=0;i<5;i++){ 
%>   //프로그래밍 코드는 스크립트릿으로 감싸줘야 한다. 

	id :<%= getId() + i %><br>  
    //화면에 출력할 코드는 표현식에 감싸져  있어야 한다.

<%
}  //프로그래밍 코드는 스크립트릿으로 감싸줘야 한다. 
%>
</body>
</html>

<%!
    String id = "u00"; //멤버변수(전역변수) 선언
    public String getId( ) { //메소드 선언
        return id;
    }
%> //전역변수, 메소드는 선언문에 감싸져 있어야 한다.
```

<br>

#### JSP 내장객체

![implicit](/img/posts/jsp/implicit.png){: width="760" height="355"}

- JSP에 입력한 코드는 대부분 _jspService() 메소드 안에 삽입되는 서블릿 코드로 생성된다. (선언문(<%! %>) 이외의 표현식(<% %>, <%= %>, <%@ %>)은 JSP가 서블릿으로 바뀌어질 때 public void _jspService 메서드 안에 코드가 작성된다.)
- _jspService()의 윗부분에 미리 선언된 객체들이 있으며, 이는 jsp에서도 사용 가능하다.
- response, request, application, session, out과 같은 변수를 내장객체라고 한다.
- 내장객체들은 _jspService() 메소드의 지역변수이므로 메소드를 벗어난 JSP 선언문에 사용될 수 없다.

<br>

### redirect
- HTTP 프로토콜로 정해진 규칙
- 서버가 브라우저에게 받은 요청을 다시 지시하는 것을 말한다.
- 서버가 클라이언트의 요청에 대해 특정 URL로 이동을 요청하는 것을 말한다.

> 1. 브라우저는 서버에게 요청을 보낸다.
> 2. 서버는 클라이언트에게 HTTP 상태코드 302로 응답하고 헤더에 Location 값이 URL을 추가하여 응답한다. (저 URL로 이동 부탁합니다.)
> 3. 클라이언트는 서버로부터 받은 상태값이 302면 Location 헤더값으로 재요청을 보내고 브라우저의 URL로 바뀌게 된다. 
> * 상태코드 302: 클라이언트가 요청한 리소스가 Location 헤더의 주어진 URL로 일시적으로 이동했음을 가리킨다. 

- 서블릿이나 JSP는 리다이렉트하기 위해 HttpServletResponse 클래스의 sendRedirect() 메소드를 사용한다.

- 예시 : 게시판에서 글 작성 - 클라이언트>서버(글 저장 요청) - 서버 글 저장 - 서버>클라이언트(글 목록으로 이동 응답) - 자동으로 글 목록(서버에게 요청하여 응답받은) 출력

![redirect](/img/posts/jsp/redirect.png){: width="760" height="416"}


##### redirect 실습
- redirect01.jsp 파일

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%
    response.sendRedirect("redirect02.jsp");
%>    
```
- redirect02.jsp 파일

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
redirect된 페이지 입니다.
</body>
</html>
```

<br>

### forward
- 같은 웹 어플리케이션 내 JSP, 서블릿이 추가적인 처리를 위해 다른 JSP, 서블릿에게 위임하여 처리하는 방식
> 1. 브라우저에서 Servlet1에게 요청을 보냄
> 2. Servlet1은 요청 처리 후 HttpServletRequest에 저장
> 3. Servlet1은 결과가 저장된 HttpServletRequest와 응답을 위한 HttpServletResponse을 같은 웹 어플리케이션 안에 있는 Servlet2에게 전송
> 4. Servlet2는 Servlet1으로 부터 받은 HttpServletRequest와 HttpServletResponse를 이용하여 요청을 처리한 후 웹 브라우저에게 결과를 전송
-  RequestDispatcher 객체는 HttpServletRequest 객체의 getRequestDispatcher(이동할 주소) 메소드를 이용해 만든다.
- RequestDispatcher의 forward() 메소드를 이용하여 요청과 응답을 넘긴다. 

![forward](/img/posts/jsp/forward.png){: width="760" height="416"}

<br>
##### forward 실습(import 부분 생략)

```java
@WebServlet("/front")
public class FrontServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
       
    /**
     * @see HttpServlet#HttpServlet()
     */
    public FrontServlet() {
        super();
        // TODO Auto-generated constructor stub
    }

    /**
     * @see HttpServlet#service(HttpServletRequest request, HttpServletResponse response)
     */
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            
            int Value = (int) 100; 
            request.setAttribute("number", Value);
            
            RequestDispatcher requestDispatehcer = request.getRequestDispatcher("/next");
            requestDispatehcer.forward(request, response);
    }

}
```

```java
@WebServlet("/next")
public class ForwardServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
       
    /**
     * @see HttpServlet#HttpServlet()
     */
    public ForwardServlet() {
        super();
        // TODO Auto-generated constructor stub
    }

    /**
     * @see HttpServlet#service(HttpServletRequest request, HttpServletResponse response)
     */
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<html>");
        out.println("<head><title>form</title></head>");
        out.println("<body>");

        int num = (Integer)request.getAttribute("number");
        out.println(num);
        
        out.println("</body>");
        out.println("</html>");
    }

}
```

<br>
### Servlet & JSP 연동
- Servlet은 java 파일, JSP는 Html 파일의 형태를 띄고 있다
- Servlet은 로직을 구현하기 알맞고, JSP은 HTML을 출력하기 알맞다.
- 보통 브라우저에서 Servlet을 요청하고 서버 내에서 Servlet이 JSP에 forward하여 요청을 처리한다.

![servletjsp](/img/posts/jsp/servletjsp
.png){: width="760" height="416"}
<br>

##### 실습(import 부분 생략)

```java
@WebServlet("/LogicServlet")
public class LogicServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
       
    /**
     * @see HttpServlet#HttpServlet()
     */
    public LogicServlet() {
        super();
        // TODO Auto-generated constructor stub
    }

    /**
     * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
     */
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        int v1 = (int)100;
        int v2 = (int)200;
        int result = v1 + v2;
        
        request.setAttribute("v1", v1);
        request.setAttribute("v2", v2);
        request.setAttribute("result", result);
        
        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/result.jsp");
        requestDispatcher.forward(request, response);
    }

}

```
- result.jsp

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>title</title>
</head>
<body>
EL표기법으로 출력합니다.<br>
${v1} + ${v2} = ${result} <br><br>

스클립틀릿과 표현식을 이용해 출력합니다.<br>
<%
    int v1 = (int)request.getAttribute("v1");
    int v2 = (int)request.getAttribute("v2");
    int result = (int)request.getAttribute("result");
%>

<%-- setAttribute로 담았던 변수를 가져온다. --%>
<!-- 이것도 주석 가능 -->

<%=v1%> + <%=v2 %> = <%=result %>
</body>
</html>
```
<BR>

- 서블릿에서 여러 개의 URL을 처리할 수 있을까?
> 폴더 경로가 일치하는 경우: /folder1/folder2/*
> 확장자 패턴이 일치하는 경우: *.html(확장자)


<br>

Reference:
- 웹 프로그래밍(풀스택)_부스트코스를 학습한 내용입니다.
- [리다이렉트란_빡통들을 위한 쉽게 재미있는 세상](https://webstone.tistory.com/65){:target="_blank"}
- [302 Found_MDN_Docs](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/302){:target="_blank"}
- [forward(), redirect()_유-잇의 개발 일지](https://u-it.tistory.com/entry/%ED%8E%98%EC%9D%B4%EC%A7%80%EC%B6%9C%EB%A0%A5-%ED%8E%98%EC%9D%B4%EC%A7%80%EC%A0%84%ED%99%98-%EB%B0%8F-%ED%8A%B9%EC%A0%95-url%EB%A1%9C-%EC%9E%AC-%EC%9A%94%EC%B2%AD-%EC%9D%84-%ED%95%B4%EC%A3%BC%EB%8A%94-RequestDispatcher-%EC%9D%98-requestgetRequestDispatcherforward-HttpServletResponse%EC%9D%98-responsesendRedirect){:target="_blank"}
- [서블릿 매핑시 url pattern 규칙_DOLOLAK](https://dololak.tistory.com/740?category=636501){:target="_blank"}
