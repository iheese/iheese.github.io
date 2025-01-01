---
layout: post
title: '[JAVA, SPRING] Spring Interceptor 내부 구현 파보기'
subtitle: 'Spring Interceptor, 내부 구현'
date: 2024-12-30 12:00:00 +0900
categories: 'SPRING'
background: '/img/posts/etc/spring.jpg'
published: false
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

                // 등록된 인터셉터가 있으면 순서대로 preHandle 메소드가 실행
                // preHandle 메소드에서 false 나오면 triggerAfterCompletion가 실행된다.
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// 핸들러 호출하여 컨트롤러 실행
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);

                // 인터셉터 postHandle 메소드 호출
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
            // 어떤 상황에서도 afterCompletion 호출
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
        // 아래 예외들이 발생해도 afterCompletion 호출
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
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

Reference:

- [19. 자동화 공격(AU)](https://lts0606.tistory.com/549)
- [[Spring Interceptor] 커스텀 어노테이션과 Intercepter 구현](https://twer.tistory.com/entry/Spring-Interceptor-%EC%BB%A4%EC%8A%A4%ED%85%80-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98%EA%B3%BC-Intercepter-%EA%B5%AC%ED%98%84)