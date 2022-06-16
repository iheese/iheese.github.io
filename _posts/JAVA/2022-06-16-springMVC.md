---
layout: post
title: '[JAVA, SPRING] SPRING MVC의 이해와 MVC 구현'
subtitle: 'Container, DispatcherServlet, Controller, HandlerMapping'
date: 2022-06-16 12:00:00 +0900
categories: 'JAVA'
background: '/img/posts/etc/spring.jpg'
---

[ 참고! [JAVA] SERVLET과 SPRING의 연결](https://ddungi.github.io/java/2022/05/17/servlet/)

- 위 컨텐츠에서 Servlet이 Spring에서 어떻게 사용되는지?, Spring MVC의  작동 방식을 대략적으로 알아보았다.
- 이번 컨텐츠는 위 컨텐츠의 연장선이라고 봐도 좋을 것 같다.

<br>

### Servlet Container와 Spring Container

![springcontainer](/img/posts/spring/springcontainer.png){: width="1000" height="500"}

1 . (실습에는 Tomcat) 서버가 실행되면 web.xml 파일에 의해 Servlet Container가 생성된다.
> - Servlet은 기본적으로 lazy-loading이므로 요청이 들어올 때 Servlet 객체가 생성된다.
> - Servlet Container는 Servlet, Listener, Filter만 생성할 수 있다.

2 . 클라이언트로부터 요청이 들어오면 Servlet Container는 DispatcherServlet를 생성한다. 

3 . DispatcherServlet은 web에서 사용되는 기본적인 Spring Container인 XmlWebApplicationContext를 생성힌다.
> - Spring Container는 pre-loading이므로 Container가 생성될 때 설정파일을 바탕으로 bean 객체와 HandlerMapping, Controller, ViewResolver을 생성한다.
> - XmlWebApplicationContext는 action-servlet.xml 을 참고한다.
> - action-servlet.xml 파일 이름 변경 가능, web.xml 파일 설정도 변경해주어야 한다.

4 . 모든 요청이 DispatcherServlet을 중심으로 들어오고 Spring Container가 아래 `작동 방식 ▼` 에 의해 요청을 처리한다.  

<br>

### Spring MVC 작동 방식

![servlet](/img/posts/javaetc/mvc.png)

1 .  클라이언트로부터 요청이 들어온다. 
2 .  생성된 DispatcherServlet이 Spring container를 생성하면서 bean 객체와 HandlerMapping, Controller, ViewResolver이 같이 생성된다.
3 .  들어온 요청에 맞는 Handler Mappping를 통해 핸들러(컨트롤러)를 조회하고 선택된 컨트롤러가 로직을 실행한다.

> - Handler Mappping을 통해  핸들러를 조회하고 Hadler Adapter를 통해 컨트롤러 로직을 실행한다.
> - Handler와 Controller가 같은 의미로 이해하면 되고 Handler가 조금 더 넓을 의미를 가지고 있다. 

4 . Controller는 DAO, DB 관련 로직을 처리한다.
5 . Controller는 DispatcherServlet에게 ModelAndView 객체(처리된 데이터(Model), 이동할 화면(View))를 리턴한다.
6 . DispatcherServlet는 ViewResolver에 ModelAndView를 전달한다.
7 . ViewResolver는 View를 생성하거나 화면 전환하여 클라이언트에게 응답한다. 

<br>

### MVC(DispatcherServlet, HandlerMapping, Controller) 구현

web.wml
- DispatcherServlet 설정 추가

```xml
<!-- 생략 -->
	<servlet>
		<servlet-name>service</servlet-name>
		<servlet-class>com.navar.web.controller.DispatcherServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>service</servlet-name>
		<url-pattern>*.service</url-pattern>
	</servlet-mapping>
<!-- 생략 -->
```

- *.service로 오는 요청은 직접 구현한 DispatcherServlet으로 모두 받는다.

DispatcherServlet.java

```java
//import 생략

public class DispatcherServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
	private HandlerMapping handlerMapping;
	
	public void init() throws ServletException {
    // DispatcherServlet가 생성되면서 HandlerMapping 객체를 생성한다. 
    handlerMapping = new HandlerMapping();
	}
	
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		//클라이언트 요청 path 정보를 추출한다. 
		String uri = request.getRequestURI();
		String path = uri.substring(uri.lastIndexOf("/"));
		
		// HandlerMapping을 통해 Controller를 얻어낸다.
		Controller controller = handlerMapping.getController(path);
		
		// 검색된 Controller 객체를 실행하고 ModelAndView 객체를 얻어낸다. 
		ModelAndView mav = controller.handleRequest(request, response);
		// Forward 방식으로 Controller가 리턴한 화면으로 이동한다.
		// ModelAndView에 저장된 검색 결과는 자동으로 request에 등록된다. 
        RequestDispatcher dispatcher = request.getRequestDispatcher(mav.getViewName());
		dispatcher.forward(request, response);
	}
}
```

HandlerMapping.java

```java
// import 생략

public class HandlerMapping {
	
	// Map 구조에 Key, Controller 형태로 객체 저장
	private Map<String, Controller> mappings;
	
	public HandlerMapping() {
		mappings = new HashMap<String, Controller>();
		
		// Controller 등록
		mappings.put("/getStudentList.service", new GetStudentListContoller());
		mappings.put("/getStudent.service", new GetStudentContoller());
		mappings.put("/insertStudent.service", new InsertStudentContoller());
		mappings.put("/updateStudent.service", new UpdateStudentContoller());
		mappings.put("/deleteStudent.service", new DeleteStudentContoller());
		
	public Controller getController(String path) {
		// mappings에 등록된 Controller 중에서 path에 해당하는 Controller 객체를 리턴한다.
		return mappings.get(path);
	}

}
```

Controller 인터페이스

```java
// import 생략 
public interface Controller {

	ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response);

}
```

Controller 구현체 예시

```java
// import 생략 

public class GetStudentListContoller implements Controller {

	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) {
		
		// 클라이언트 입력정보 추출
		String searchCondition = request.getParameter("searchCondition");
		String searchKeyword = request.getParameter("searchKeyword");

		// Null Check
		if(searchCondition == null) searchCondition = "NAME";
		if(searchKeyword == null) searchKeyword = "";
		
		request.setAttribute("condition", searchCondition);
		request.setAttribute("keyword", searchKeyword);

		// DB 연동 처리
		StudentVO vo = new StudentVO();
		vo.setSearchCondition(searchCondition);
		vo.setSearchKeyword(searchKeyword);
		
		StudentDAO dao = new StudentDAOJDBC();
		List<StudentVO> studentList = dao.getStudentList(vo);
		
		//ModelAndView에 Model과 View 정보를 입력하고 리턴한다.
		ModelAndView mav = new ModelAndView();
        mav.addObject("studentList",studentList); //Model 정보
        mav.setViewName("getStudentList.jsp"); //View 정보
        return mav;
	}
}

```

- 위 방법은 DispatcherServlet, HandlerMapping, Controller 객체를 직접 생성하여 MVC의 로직을 구현한 것이다.
- 이제 Spring이 제공하는 DispatcherServlet, HandlerMapping와 Controller를 사용해보자 

<br>

### Spring MVC(DispatcherServlet, HandlerMapping, Controller) 

- 아래 방법은 xml 파일 설정만으로 Spring이 제공하는 DispatcherServlet, HandlerMapping와 Controller를 사용하는 방법이다. 

web.xml

```xml
<!-- 생략 -->
  <servlet>
    <servlet-name>service</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
    	<param-name>contextConfigLocation</param-name>
    	<param-value>/WEB-INF/action-servlet.xml</param-value>
    </init-param>
  </servlet>
  
  <servlet-mapping>
    <servlet-name>service</servlet-name>
    <url-pattern>*.service</url-pattern>
  </servlet-mapping>
<!-- 생략 -->
```

- *.servlet으로 들어오는 모든 요청은 Spring이 제공하는 DispatcherServlet으로 받는다.
- DispatcherServlet는 /WEB-INF/action-servlet.xml을 참고하여 스프링 컨테이너를 생성하겠다는 의미이다. (param-name은 고정)

action-servlet.xml

```xml
<!--생략-->
<!-- Controller 객체 등록-->
<bean id="getStudentList" class="com.naver.web.controller.student.GetStudentListController"/>
	<bean id="getStudent" class="com.naver.web.controller.student.GetStudentController"/>
	<bean id="deleteStudent" class="com.naver.web.controller.student.DeleteStudentController"/>
	<bean id="insertStudent" class="com.naver.web.controller.student.InsertStudentController"/>
	<bean id="updateStudent" class="com.naver.web.controller.student.UpdatStudentController"/>	

<!-- HandlerMapping 객체 등록-->
	<bean id="handlerMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
	<property name="mappings">
		<props>
			<prop key="/getStudentList.service">getStudentList</prop>
			<prop key="/getStudent.service">getStudent</prop>
			<prop key="/deleteStudent.service">deleteStudent</prop>
			<prop key="/insertStudent.service">insertStudent</prop>
			<prop key="/updateStudent.service">updateStudent</prop>
			</props>
	</property>
</bean>
    
<!--생략-->
```

- Spring Container가 사용할 수 있게 Controller 객체, HandlerMapping 객체를 등록해주어야 한다. 

<br>

### 문제점
- Controller를 구현할 때 지켜야 할 점이 많아 코드 작성이 까다로워진다.
- Controller의 수가 많아질수록 파일의 관리가 어려워지고 .xml 파일의 유지보수가 어려워진다.

<hr>

- 위 문제는 Annotation, ViewResolver를 이용해 해결할 수 있다.

<br>
Reference:
- [inflean_img](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC_renew)
- [Servlet Container and Spring Framework img_Moss GU](https://mossgreen.github.io/Servlet-Containers-and-Spring-Framework/)
- [Spring(8) - HandlerMapping, HandlerAdapter_하면된다](https://the-greatest-developer.tistory.com/13)
- [[spring] HandlerMapping, HandlerAdapter, HandlerInterceptor_기록은 기억의 연장선](https://joont92.github.io/spring/HandlerMapping-HandlerAdapter-HandlerInterceptor/)
