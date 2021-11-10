---
layout: post
title: 'HTML '
subtitle: 'HTML_ 표 만들기'
date: 2021-11-05 12:00:00 +0900
categories: 'HTML'
use_math: true
---

```html
<!DOCTYPE html> <!--HTML파일 선언-->
<html>
<head>
  <title>Ship To It - Company Packing List</title>
  <link href="https://fonts.googleapis.com/css?family=Lato: 100,300,400,700|Luckiest+Guy|Oxygen:300,400" rel="stylesheet">
  <link href="style.css" type="text/css" rel="stylesheet">
</head>
<body>

  <ul class="navigation">
    <li><img src="https://content.codecademy.com/courses/web-101/unit-9/htmlcss1-img_logo-shiptoit.png" height="20px;"></li>
    <li class="active">Action List</li>
    <li>Profiles</li>
    <li>Settings</li>
  </ul>

  <div class="search">Search the table</div>
  <table> <!--표 만들기 선언-->
    <thead> <!--표의 헤드-->
    <tr>  <!--행 별로 구성-->
      <th scope="col">Company Name</th> <!--열을 위한 헤드임을 선언-->  
      <th scope="col">Number of Items to Ship</th>
      <th scope="col">Next Action</th>
    </tr>
    </thead>
  <tbody> <!--표의 본격적 내용 선언-->
    <tr>
      <td colspan="2">Adam’s Greenworks</td> <!--colspan 열 2개 차지-->
      <td>14</td>
      <td>Package Items</td>  <!--표의 데이터를 넣을 때는 <td>-->
    </tr>
    <tr>
  <td rowspan="2">Davie's Burgers</td> <!--rowspan 행 2개 차지-->
  <td>2</td>
  <td>Send Invoice</td>
</tr>
<tr>
  <td>Baker's Bike Shop</td>
  <td>3</td>
  <td>Send Invoice</td>
</tr>
<tr>
  <td>Miss Sally's Southern</td>
  <td>4</td>
  <td>Ship</td>
</tr>
<tr>
  <td>Summit Resort Rentals</td>
  <td>4</td>
  <td>Ship</td>
</tr>
<tr>       <!--각 행별로 데이터를 입력 하는 것이다. -->
  <td>Strike Fitness</td>
  <td>1</td>
  <td>Enter Order</td>
</tr>
  </tbody>
  <tfoot>    
    <td>Total</td>
    <td>28</td>  <!--테이블의 foot contents를 묶을 때 사용 -->
  </tfoot>
  </table>
</body>
</html>
```







Reference:

Learn HTML_Codecademy를 바탕으로 공부하는 내용입니다. 