---
layout: post
title: 'HTML 기본 문법 및 문법'
subtitle: 'HTML_ 기본 구성, id 선언 등'
date: 2021-11-04 12:00:00 +0900
categories: 'FrontEnd'
use_math: true
---

```html
<body>
  <h1>The Brown Bear</h1> <!-- 각주 -->
  <div id="introduction">
    <h2>About Brown Bears</h2>  <!-- h1~h6 헤드라인 제목 설정 가능 -->
    <p>The brown bear (<em>Ursus arctos</em>) is native to parts of northern Eurasia and North America. Its conservation status is currently <strong>Least Concern</strong>.<br /><br /> There are many subspecies within the brown bear species, including the Atlas bear and the Himalayan brown bear.</p>
    <h3>Species</h3>        <!-- <p>,<div>,<span> 은 텍스트나 블럭을 구분할 때 사용 -->
    <ul>					<!-- <em>:이탈릭체 <strong>: 볼드체-->
      <li>Arctos</li>  			<!-- <br>은 줄 띄기 -->
      <li>Collarus</li>   
      <li>Horribilis</li>
      <li>Nelsoni (extinct)</li> <!-- 순서없는 목록-->
    </ul>
    <h3>Features</h3>
    <p>Brown bears are not always completely brown. Some can be reddish or yellowish. They have very large, curved claws and huge paws. Male brown bears are often 30% larger than female brown bears. They can range from 5 feet to 9 feet from head to toe.</p>
  </div>
  <div id="habitat"> 
    <h2>Habitat</h2>
    <h3>Countries with Large Brown Bear Populations</h3>
    <ol>
      <li>Russia</li>
      <li>United States</li>
      <li>Canada</li>   <!-- 순서있는 목록화 -->
    </ol>
    <h3>Countries with Small Brown Bear Populations</h3>
    <p>Some countries with smaller brown bear populations include Armenia, Belarus, Bulgaria, China, Finland, France, Greece, India, Japan, Nepal, Poland, Romania, Slovenia, Turkmenistan, and Uzbekistan.</p>
  </div>
  <div id="media">  <!-- img, video도 넣을 수 있음 -->
    <h2>Media</h2>
    <img src="https://content.codecademy.com/courses/web-101/web101-image_brownbear.jpg" alt="A Brown Bear"/>
    <video src="https://content.codecademy.com/courses/freelance-1/unit-1/lesson-2/htmlcss1-vid_brown-bear.mp4" width="320" height="240" controls>Video not supported</video>
  </div>
</body>
```

```html
<!DOCTYPE html>  <!--HTML 파일이라고 선언하는 것-->
<html> <!--HTML 코드 포함-->

<head> <!--제목 같은 웹페이지 정보를 넣을 수 있다.  -->
  <title>Brown Bears</title> <!-- <title>을 사용하여 제목을 구성 웹페이지의 탭에 뜬다. -->
</head>

<body>
  <nav>
    <a href="./index.html">Brown Bear</a>  <!--앵커 요소는 내부 페이지, 외부 페이지, 컨텐츠에 연결해주는 역할을 한다. -->
    <a href="./aboutme.html">About Me</a>
  </nav>
  <h1>The Brown Bear</h1>
  <nav>
    <ul>
      <li><a href="#introduction">Introduction</a></li> <!--introduction으로 점프!-->
      <li><a href="#habitat">Habitat</a></li>
      <li><a href="#media">Media</a></li>
    </ul>
  </nav>
  <div id="introduction"> <!--id를 선언하고 나중에 #을 붙여서 이 위치로 점프할 수 있다. -->
    <h2>About Brown Bears</h2>
    <p>The brown bear (<em>Ursus arctos</em>) is native to parts of northern Eurasia and North America. Its conservation status is currently <strong>Least Concern</strong>.<br /><br /> There are many subspecies within the brown bear species, including the
      Atlas bear and the Himalayan brown bear.</p>
    <a href="https://en.wikipedia.org/wiki/Brown_bear" target="_blank">Learn More</a>
    <h3>Species</h3>
    <ul>
      <li>Arctos</li>
      <li>Collarus</li>
      <li>Horribilis</li>
      <li>Nelsoni (extinct)</li>
    </ul>
    <h3>Features</h3>
    <p>Brown bears are not always completely brown. Some can be reddish or yellowish. They have very large, curved claws and huge paws. Male brown bears are often 30% larger than female brown bears. They can range from 5 feet to 9 feet from head to toe.</p>
  </div>
  <div id="habitat">
    <h2>Habitat</h2>
    <h3>Countries with Large Brown Bear Populations</h3>
    <ol>
      <li>Russia</li>
      <li>United States</li>
      <li>Canada</li>
    </ol>
    <h3>Countries with Small Brown Bear Populations</h3>
    <p>Some countries with smaller brown bear populations include Armenia, Belarus, Bulgaria, China, Finland, France, Greece, India, Japan, Nepal, Poland, Romania, Slovenia, Turkmenistan, and Uzbekistan.</p>
  </div>
  <div id="media">
    <h2>Media</h2>
    <img src="https://content.codecademy.com/courses/web-101/web101-image_brownbear.jpg" />
    <video src="https://content.codecademy.com/courses/freelance-1/unit-1/lesson-2/htmlcss1-vid_brown-bear.mp4" height="240" width="320" controls>Video not supported</video>
  </div>
</body>

</html>

<!--1. 공백은 HTML 가독성에 도움을 준다
    2. 띄어쓰기는 HTML 가독성에 도움을 준다. -->
```



<BR>

Reference:

Learn HTML_Codecademy를 바탕으로 공부하는 내용입니다. 