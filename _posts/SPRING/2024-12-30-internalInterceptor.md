---
layout: post
title: '[JAVA, SPRING] Spring Interceptor 중심으로 DispatcherServlet 내부 구현 파보기'
subtitle: 'Spring Interceptor, 내부 구현, DispatcherServlet, doDispatch'
date: 2024-12-30 12:00:00 +0900
categories: 'SPRING'
background: '/img/posts/etc/spring.jpg'
published: true
---

# Spring Interceptor

```java
public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception {

    return true;
    }

    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
    @Nullable ModelAndView modelAndView) throws Exception {
    }  

	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, 
    
    @Nullable Exception ex) throws Exception {
	}
}
```


- Sprint Interceptor를 사용하기 위해 구현해야 할 HandlerInterceptor 인터페이스이다. 

![interceptor](https://github.com/user-attachments/assets/99546c16-bbe1-4fc6-a94d-bffdc61e8016)


- 대략적으로 위의 사진처럼 디스패처서블릿이 컨트롤러에 요청을 전달하기 전,후에 작동하는 것으로 알고 있었다.
- preHandler는 컨트롤러에 요청이 들어가기 전에, postHandler는 컨트롤러에서 요청이 처리되고 난 뒤에,
 afterCompletion은 ViewResolver를 통해 View 까지 만들어지고 난 뒤 요청 처리가 완료된 뒤 실행되는 것으로 알고 있었다.

 <br>

# 좀 더 깊이 DispatcherServlet으로!

```java
public class DispatcherServlet extends FrameworkServlet {

    //...

    	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// 여기서 핸들러를 조회
                // 핸들러 없으면 NoHandlerFoundException을 터트리거나, 404 Error를 보낸다.
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}
				
				// 아래 내부 코드 확인!!!
                // Object > HandlerAdapter 리턴
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				String method = request.getMethod();
				boolean isGet = HttpMethod.GET.matches(method);
				if (isGet || HttpMethod.HEAD.matches(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				// 아래 내부 코드 확인!!!
                // 등록된 인터셉터가 있으면 순서대로 preHandle 메소드가 실행
                // preHandle 메소드에서 false 처리되어도 postHandle은 실행되지 않지만 내부적으로 triggerAfterCompletion가 실행된다.
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// 핸들러 호출하여 컨트롤러 실행
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
 
				// 이레 부분은 비동기 요청 처리를 위해 사용된다.
				// true면 비동기 모드이므로, 동기 방식으로 처리하지 않도록 실행을 중단시킨다. 
				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				// modelAndView 에 View 등록
				applyDefaultViewName(processedRequest, mv);

				// 아래 내부 코드 확인!!!
                // 인터셉터 postHandle 메소드 호출
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}

			// 아래 내부 코드 확인!!!
			// 요청 처리가 끝난 후 응답을 클라이언트에게 전송하기 위해 호출되는 부분
			// 컨트롤러 처리 후 결과를 처리하거나, 발생한 예외를 처리하는 역할을 한다.
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
        // 예외나 에러가 발생해도 afterCompletion 호출
		catch (Exception ex) {
			// 아래 내부 코드 확인!!
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			// 비동기 처리일 경우
			if (asyncManager.isConcurrentHandlingStarted()) {
				// 아래 내부 코드 확인!!!
				// 비동기 처리일 경우 후처리 메소드 호출
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// 멀티파트 요청일 경우 리소스 해제(임시 파일 삭제, 메모리 정리) 작업 수행
				// 멀티파트 요청: 보통 파일 업로드와 같은 작업 
				// 헤더에 Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryXYZ 형태로 포함되어있다.
				// boundary로 여러 파트로 구분된다. 
				// 폼 데이터와 파일 데이터(바이너리 데이터)를 모두 포함할 수 있다.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}

    //...
}
```

<br>

## getHandlerAdapter

```java
	protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		if (this.handlerAdapters != null) {
			for (HandlerAdapter adapter : this.handlerAdapters) {
				if (adapter.supports(handler)) {
					return adapter;
				}
			}
		}
		throw new ServletException("No adapter for handler [" + handler +
				"]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
	}
```

- 핸들러 목록에서 맞는 핸들러를 찾는 메소드

<br>

## applyPreHandle

```java
	boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
		for (int i = 0; i < this.interceptorList.size(); i++) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			if (!interceptor.preHandle(request, response, this.handler)) {
				triggerAfterCompletion(request, response, null);
				return false;
			}
			this.interceptorIndex = i;
		}
		return true;
	}
```

- 여러 개의 인터셉터가 있으면 preHandle이 등록 순서대로 실행되는 것을 알 수 있다.
- preHandle 메소드에서 false 가 나오면 afterCompletion을 실행하는 triggerAfterCompletion 메소드가 실행된다.

<br>

## applyPostHandle

```java
	void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv)
			throws Exception {

		for (int i = this.interceptorList.size() - 1; i >= 0; i--) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			interceptor.postHandle(request, response, this.handler, mv);
		}
	}
```

- 여러 개의 인터셉터가 있으면 postHandle이 등록 역순대로 실행되는 것을 알 수 있다.

<br>

## processDispatchResult

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
			@Nullable Exception exception) throws Exception {

		boolean errorView = false;

		// 예외가 있을 경우 처리
		if (exception != null) {
			if (exception instanceof ModelAndViewDefiningException) {
				logger.debug("ModelAndViewDefiningException encountered", exception);
				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
			}
			else {
				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
				mv = processHandlerException(request, response, handler, exception);
				errorView = (mv != null);
			}
		}

		//  예외가 없는 경우 ModelAndView 렌더링
		if (mv != null && !mv.wasCleared()) {
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
		else {
			if (logger.isTraceEnabled()) {
				logger.trace("No view rendering, null ModelAndView returned.");
			}
		}

		// 비동기일 경우 처리 마무리
		if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
			// Concurrent handling started during a forward
			return;
		}

		// afterCompletion 호출
		if (mappedHandler != null) {
			// Exception (if any) is already handled..
			mappedHandler.triggerAfterCompletion(request, response, null);
		}
	}
```

<br>

## triggerAfterCompletion

```java
void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex) {
		for (int i = this.interceptorIndex; i >= 0; i--) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			try {
				interceptor.afterCompletion(request, response, this.handler, ex);
			}
			catch (Throwable ex2) {
				logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
			}
		}
	}
```

- 여러 개의 인터셉터가 있으면 afterCompletion이 등록 역순대로 실행되는 것을 알 수 있다.

<br>

## applyAfterConcurrentHandlingStarted

```java
	void applyAfterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response) {
		for (int i = this.interceptorList.size() - 1; i >= 0; i--) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			if (interceptor instanceof AsyncHandlerInterceptor) {
				try {
					AsyncHandlerInterceptor asyncInterceptor = (AsyncHandlerInterceptor) interceptor;
					asyncInterceptor.afterConcurrentHandlingStarted(request, response, this.handler);
				}
				catch (Throwable ex) {
					if (logger.isErrorEnabled()) {
						logger.error("Interceptor [" + interceptor + "] failed in afterConcurrentHandlingStarted", ex);
					}
				}
			}
		}
	}
```

- AsyncHandlerInterceptor을 구현한 인터셉터에 대해 비동기 처리가 시작되었음을 알리는 afterConcurrentHandlingStarted 메소드 호출
- 비동기 작업이 시작된 이후에 등록된 모든 인터셉터의 후처리 메소드(postHandle, afterCompletion)를 실행하기 위해 호출
- 여러 개의 인터셉터가 있으면 후처리 메소드를 처리하기 떄문에 afterConcurrentHandlingStarted 메소드가 등록 역순대로 실행되는 것을 알 수 있다.

<br>

# AsyncHandlerInterceptor

```java
public interface AsyncHandlerInterceptor extends HandlerInterceptor {

	default void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response,
			Object handler) throws Exception {
	
}
```

- 비동기 처리용 Interceptor 인터페이스 
- 비동기 요청 순서
> - 비동기 요청 들어오면 preHandle 
> - 비동기 처리 시작 afterConcurrentHandlingStarted
> - 요청 완료 후 afterCompletion
- 동기 요청일 때만 postHandle이 처리된다. 비동기 요청에서 후처리가 필요하면 afterConcurrentHandlingStarted 이나 afterCompletion을 사용하면 된다. 

<br>

Reference:

- [스프링 공식 문서 HandlerInterceptor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/HandlerInterceptor.html)