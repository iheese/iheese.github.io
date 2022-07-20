---
layout: post
title: '[ERROR, HTML] HTML, JSP에서 form 태그 안에 button 태그 에러'
subtitle: 'form 태그 안에서 작동하는 button 태그, button 태그 타입 설정'
date: 2022-07-20 12:00:00 +0900
categories: 'ERROR'
background: '/img/posts/etc/error.jpg'
---

### 에러 상황

- 댓글 삭제, 댓글 등록 기능을 구현하면서 첫 번째 클릭(삭제 버튼, 등록 버튼)은 아무 반응이 없고 삭제, 등록은 기능은 실행이 되었는데 화면 전환이 안되었다.
- 더 이상한 건 두 번째 클릭부터는 잘 작동했다.

<br>

### 수정 전 코드

```html
<form>
		<input type="hidden" id="id" value="${post.id}" />
		<button id="btn-reply" class="btn btn-secondary">덧글등록</button>
</form>
				
```

<br>

### 원인
- form 태그 안에 button 태그가 있어서 그랬다.
- button 태그의 기본값은 submit이라서 type 설정을 해주지 않으면 url을 통해 값을 제출하려 한다.
- 어쩐지 자꾸 url 뒤에 ? 가 붙어서 파라미터를 기다리고 있었다.

<br>

### 수정 후 코드

```html
<form>
		<input type="hidden" id="id" value="${post.id}" />
		<button type="button" id="btn-reply" class="btn btn-secondary">덧글등록</button>
        <!--변경부분-->
</form>
				
```

```html
<form>
		<input type="hidden" id="id" value="${post.id}" />
</form>
<button id="btn-reply" class="btn btn-secondary">덧글등록</button> 
<!--변경부분-->
				
```

- button 태그에 type="button"을 추가해주는 방법이 있다.
- button 태그를 form 태그에서 빼주는 방법이 있다.

<hr>

- 첫 번째 클릭에서는 안되다가 다음 클릭부터 기능이 잘 구현되었던 것은 첫 번째 클릭에서 파라미터를 요청하는 동시에 js파일에 연결되었기 때문인 것 같다. 
- 그래서 첫 번째는 기능만 수행되고 다음 부터는 잘 연결되어 실행되었던 것 같다.

<br>

### 마무리
- form 내 값을 제출할 용도가 아닌 js 파일과 연결해서 사용할 예정이라면 type="button"을 꼭 정해주어야 겠다.
- HTML 도 공부할 게 많고 버튼에 대해 이해가 아직 많이 부족한 것 같다.

<br>

Reference:
- [error img _ fgStudy](https://velog.io/@dev-redo/NextJS-Image-Tag-Error-Invalid-src-prop)
- [`<button>`_mdn web docs](https://developer.mozilla.org/ko/docs/Web/HTML/Element/button)
- [`<form>`_mdn web docs](https://developer.mozilla.org/ko/docs/Web/HTML/Element/form)