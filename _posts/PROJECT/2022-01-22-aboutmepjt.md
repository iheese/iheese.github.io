---
layout: post
title: "나를 소개하는 홈페이지"
subtitle: "부스트코스 프로젝트 A"
date: 2022-01-27 12:00:00 +0900
categories: "PROJECT"
---

### For 나를 소개하는 홈페이지

- [HTML ,Basic CSS](https://ddungi.github.io/frontend/2022/01/26/basiccss/)
- [Web 개발의 이해](https://ddungi.github.io/web/2022/01/17/webprogramming1/)
- [Servlet](https://ddungi.github.io/java/2022/01/27/servlet/)

<br>

#### 웹프론트엔드 기술요구사항

- html layout tag를 사용합니다.
- classname은 일정한 컨벤션을 유지합니다.
- 의미에 맞는 tag를 최대한 사용합니다. (div 사용은 최대한 자제)
- position속성과 float를 사용해서 element를 배치합니다.
- 라이브러리를 사용한 레이아웃은 지양합니다. (부트스트랩 등)
- id와 class등의 다양한 selector문법을 잘 활용해야 합니다.


#### 웹백엔드 기술요구사항

- 톰캣서버를 통해서 자기소개 페이지가 동작되야 합니다. (ex http://localhost:8080/aboutme/index.html 에서 노출)
- 서블릿페이지하나를 생성해서, url을 입력했을 때 시간데이터가 화면에 노출돼야 합니다.
<br>

*eclipse에서 프로젝트가 진행되었습니다.

*톰캣 8.5 버전을 사용했습니다. 
<br>

![index](/img/posts/aboutmeproject/index.png){: width="800" height="374"}

- 네비게이션 바(홈, 자기소개, 내 사진, 몇 시예요) 구성
- 홈 네비게이션(자기소개, 내 사진) 구성
- footer 구성
<br>

![aboutme](/img/posts/aboutmeproject/aboutme.png){: width="800" height="374"}
<br>

- 세 가지 질문 밑에 답변 HTML 구성
- 포트폴리오 링크 구성

![photo](/img/posts/aboutmeproject/photo.png){: width="800" height="374"}
<br>

- 사진 두 개와 날짜, 설명 구성

![today](/img/posts/aboutmeproject/today.png){: width="800" height="374"}
<br>

- 톰캣 서버를 통해 현재 시간 출력 
- 톰캣 서버를 통해서 자기 소개  페이지 연결
- 메인 화면과 연결되는 링크 구성

<br>

#### 문제점
1 . 톰캣서버를 이용해 실행했을 때 메인 화면으로 연결하는 과정
 - [Tomcat 서버 연동 해서 index.html 페이지 띄우기_북항](https://webfirewood.tistory.com/5) 를 참고하여 만든 HTML 파일과 CSS 파일을 /src/main/webapp 로 옮겨줬다. 

? WebContent 파일에 직접 연결할 수 없는가?
> - [WebContent와 webapp 차이점_ ds](https://ehdtnn.tistory.com/773) 를 참고
> - 오른쪽 클릭 project Properties - Deployment Assembly - Source - Add: /WebContent - Deploy Path: /  
> - WebContent에 HTML, CSS 파일 두고 서버 실행!


2 . 서버를 통해 HTML, css 파일을 올렸을 때 깨지는 현상
- png 이미지 파일은 인식하지 못하고 jfif 파일은 인식하는 현상
> - 이미지 변환으로 바꿔주었다. (.png -->.jfif)
- photo.css 값이 설정했던 값과 다르게 들어갔다.
> - 서버 연결한 상태로 photo.css 수정 


<br>

Reference:
- 나를 소개하는 홈페이지_부스트 코스 프로젝트를 진행한 과정입니다.
- 동료 학습자의 원활한 학습을 위해 소스 코드 업로드를 하지 않았습니다.

