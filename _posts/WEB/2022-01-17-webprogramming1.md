---
layout: post
title: '웹 프로그래밍(풀스택)'
subtitle: 'Web 개발의 이해'
date: 2022-01-17 11:00:00 +0900
categories: 'WEB'
use_math: true
---

### HTTP(Hypertext Transfer Protocol)

- 서버와 클라이언트가 인터넷 상에서 데이터를 주고받기 위한 프로토콜
 > 장점: 
 > - 불특정 다수를 대상으로 하는 서비스에 적합.
 > - 클라이언트와 서버가 계속 연결된 상태가 아니므로 클라이언트와 서버 간의 최대 연결 수보다 많은 요청과 응답을 처리할 수 있다.

 > 단점:
 > - 연결을 끊어버리기 때문에 클라이언트의 이전 상황을 알 수 없다.
 > - 이러한 특징을 무상태(Stateless)라 한다.
 > - 이러한 특징으로 인해 Cookie 같은 기술이 등장하게 되었다.


#### URL(Uniform Resource Locator)
- 인터넷 상의 자원의 위치
- 특정 웹 서버의 특정 파일에 접근하기 위한 경로 혹은 주소

![http](/img/posts/webprogramming/http.png)

- 요청 메소드: GET, PUT, POST 등의 요청 방식
> - GET: 정보를 요청한다(SELECT)
> - POST: 정보를 밀어넣는다.(INSELT)
> - PUT: 정보를 업데이트한다.(UPDATE)
> - DELETE: 정보를 삭제한다.(DELETE)
> - HEAD: HTTP 헤더 정보만 요청, 해당 자원의  존재 여부, 서버의 문제를 확인할 때 사용한다.
> - OPTIONS: 웹서버가 지원하는 메소드 종류 요청
> - TRACE: 클라이언트의 요청 반환,echo 서비스로 서버 상태 확인할 때 사용한다. 
- 요청 URL: 요청하는 자원의 위치 명시
- HTTP 프로토콜 버전: 웹 브라우저가 사용하는 프로토콜 버전
<BR>

### 웹 프론트엔드
- 사용자에게 웹을 통해 다양한 콘텐츠 제공
- 사용자의 요청에 따라 반응하고 동작한다.
> - 웹 컨텐츠를 잘 보여주기 위한 구조: HTML
> - 적절한 배치와 일관된 디자인(보기 좋게): CSS
> - 사용자의 요청을 잘 반영해야 함(소통하듯이): Javascript

### 웹 백엔드
- 정보를 처리하고 저장하며, 요청에 따라 정보를 내려주는 역할

### 브라우저(Browser)
- WWW(World Wide Web)에서 정보를 검색, 표현, 탐색하기 위한 소프트웨어
- 특정 정보로 이동할 수 있는 주소 입력창이  있고, HTTP를 이용해 정보를 주고 받을 수 있는 네트워크 모듈도 포함하고 있다.
- 서버에서 받은 문서(HTML, CSS, Javascript)를 해석하고 실행하여 화면에 표시하기 위한 해석기(Parser)를 가지고 있다. 

![webkit](/img/posts/webprogramming/webkitflow.png)

사파리 브라우저에서 처리되는 webkit렌더링엔진의 처리과정

- HTML를 해석해 DOM Tree를 만들고, CSS를 해석해 CSS Tree를 만듭니다.
- 이 과정에서 화면에 표시하기 위해 parsing 과정이 필요하다.
- DOM Tree + CSS Tree >> Render Tree >>> Painting >> Display
<br>

### 웹 서버(Web Server)
- 웹 서버는 소프트웨어를 말하지만, 웹 서버 소프트웨어가 동작하는 컴퓨터를 말함
- 클라이언트가 요청하는 HTML 문서나 각종 리서스 전달하는 역할
- 리소스의 대상은 정적인 데이터이거나 동적인 결과가 될 수 있다.
- 대표적인 예: Apache, Nginx, Microsoft llS 등

#### DBMS(DataBase Management System)
- 다수의 사용자가 데이터베이스 내의 데이터에 접근할 수 있도록 해주는 소프트웨어

#### 미들웨어(Middle Ware)
- 클라이언트 쪽에 비지니스 로직이 많을 경우 클라이언트 관리로 인해 과비용 문제가 발생한다.
- 비지니스 로직을 클라이언트와 DBMS 사이의 미들웨어 서버에서 동작하도록 함으로써 클라이언트는 입출력만 담당하게 부담을 줄여주는 방식


![was](/img/posts/webprogramming/was.png)

#### WAS(Web Application Server)
- WAS는 일종의 미들웨어로 웹 클라이언트의 요청 중 웹 애플리케이션이 동작하도록 지원하는 목적을 가짐
- WAS도 자체적으로 웹 서버 기능을 내장함
- 현재 WAS도 정적인 컨텐츠를 처리하는 데 있어서 성능상 웹 서버와 차이가 없음.
- 규모가 커질수록 웹 서버와 WAS를 분리함
- 자원 이용의 효율성 및 장애 극복, 배포 및 유지보수의 편의성을 위해 웹서버(정적 컨텐츠)와 WAS(동적 컨텐츠)를 분리함

<BR>

Reference:
- 웹 프로그래밍(풀스택)_부스트코스를 학습한 내용입니다.
- [webkitflow](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)
