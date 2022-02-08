---
layout: post
title: "JSP와 SERVLET"
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

### forward

### Servlet & Jsp 연동



<br>
Reference:
- 웹 프로그래밍(풀스택)_부스트코스를 학습한 내용입니다.
