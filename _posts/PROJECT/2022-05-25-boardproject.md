---
layout: post
title: 'JSP BOARD PROJECT 회고'
subtitle: 'JSP, Servlet, JDBC, H2, MVC, EL, JSTL, Session '
date: 2022-05-26 12:00:00 +0900
categories: 'PROJECT'
background: '/img/posts/project/project.jpg'
---

### JSP를 이용한 게시판 프로젝트입니다.

- #### [ 어떤 프로젝트인지 자세한 내용 + 소스 코드 클릭!! ](https://github.com/ddungi/MyJSPBoard)

<br>

## JSP와 Servlet으로 게시판 만들기 과정

1. Servlet으로 모든 동적 페이지의 로직과 view까지 구현했으며 Html 파일로 데이터를 받기만 했다. 
2. Model1 architecture : JSP를 이용하여 Controller(사용자 입력 추출, DB 연동 처리, 화면 이동)와 view를 합치고 Servlet으로 주요 로직과 DB 연동을 처리하게 하였다.   
3. Model2 architecture : MVC(Model, View, Controller) 모델을 적용하여 JSP 파일의 Controller 부분을 DispatcherServlet로 만들어 주요 로직와 DB 연동을 처리했다. JSP는 View만 만드는 역할로 만들었다.
4. JSP의 내부의 코드에 EL(Expression Language) , JSTL(JavaServer Pages Standard Tag Library)을 이용해 가독성을 좋게 했다.

<br>

#### 어려웠던 점 및 더 필요한 점
- 회원 가입시 유효성 검사 기능
- 더 많은 게시판 생성(메뉴 이동)
- 디렉토리가 점점 복잡해질수록 이해하는 것에 어려움이 있었다. 

<br>

Reference:
- [magenest img](https://magenest.com/en/project-management-software/)
  


