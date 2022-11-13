---
layout: post
title: '[JAVA, SPRINGBOOT] JUNIT, REST API + Annotations'
subtitle: 'Test case, @GetMappping, @PostMapping, @PutMapping, @DeleteMapping, @RequestBody, @ResponseBody, etc'
date: 2022-07-08 12:00:00 +0900
categories: 'SPRING'
background: '/img/posts/etc/spring.jpg'
---

- 모든 실습은 Eclipse Spring Tool Suite 4 에서 진행되었습니다.

<br>

### JUnit
- Java Programming 단위 테스트 프레임워크이다. 
- 매번 서버를 구동하여 브라우저를 통해 테스트하는 과정은 매우 복잡하고 귀찮기 때문에 JUnit이 필요하다.
- 일반 Ecilpse에서 JUnit을 사용하는 방법이다.
> - 프로젝트 오른쪽 클릭 - Properties - Java Build Path - Libraries - Add Library - JUnit 선택, 버전 선택 - finish

- Spring Boot나 Maven 빌드를 사용한다면 아래와 같은 의존성을 추가해주면 된다. 

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
```

<br>

### Test Case 만들기, 실습

- Spring Starter Project 로 프로젝트를 만들면 테스트를 위한 준비가 모두 되어 있다.
- src/main/java에 테스트할 클래스를 생성하고 src/main/test에 테스트 파일을 만들면 된다.

<br>

- src/main/java/패키지에 계산기 기능의 클래스를 생성했다.

Calc.java

```java
public class Calc {
	private int a;
	private int b;
	
	public Calc(int a, int b) {
		this.a = a;
		this.a = b;
	}
	
	public int plus() {
		return a+b;
	}
	
	public int minus() {
		return a-b;
	}
	public int multiply() {
		return a*b;
	}
	public int divide() {
		return a/b;
	}
}

```

- src/main/test/패키지에서 New - JUnit Test Case를 선택한다.
- 테스트 클래스 명을 정하고 Annotation이 붙은 메소드를 선택할 수 있다.
> -  Which method stubs would you like to create?
> > @BeforeAll : 해당 테스트 클래스를 초기화할 때 한 번만 실행된다. (static 필수)
> > @AfterAll :  해당 테스트 클래스 내 모든 테스트 메소드가 실행된 후 한 번만 실행된다. (static 필수)
> > @BeforeEach : 매번 각 테스트 메소드 전에 실행된다.
> > @AfterEach : 매번 각 테스트 메소드 이후에 실행된다.
> > @Test : 테스트가 일어나는 메소드이다.
> > @Disabled : Annotation이 붙은 테스트 메소드는 무시된다.

- 테스트 순서: 
@BeforeAll - (@BeforeEach - @Test - @AfterEach) x 여러번 - @AfterAll
- Class under test 에서 내가 테스트할 클래스를 선택한다. 
- Next를 누르면 클래스 내 테스트할 메소드를 선택할 수 있다.

CalcTest.java

```java
class CalcTest {
	private Calc calc;	
	
	@BeforeAll
	static void setUpBeforeClass() throws Exception {
		System.out.println("@BeforeAll 실행");
	}

	@AfterAll
	static void tearDownAfterClass() throws Exception {
		System.out.println("@AfterAll 실행");
	}

	@BeforeEach
	void setUp() throws Exception {
		System.out.println("@BeforeEach 실행");
		calc=new Calc(6,2);
	}

	@AfterEach
	void tearDown() throws Exception {
		System.out.println("@AfterEach 실행");
	}

	@Test
	void testPlus() {
		assertEquals(8,calc.plus());
	}

	@Test
	void testMinus() {
		assertEquals(4,calc.minus());
	}

	@Test
	void testMultiply() {
		assertEquals(12,calc.multiply());
	}

	@Test
	void testDivide() {
		assertEquals(3,calc.divide());
	}

}
```

- JUnit 찾에 초록색 줄이 뜨면 모든 테스트가 성공이고, 빨간 줄이 뜨면 실패한 테스트가 있는 것이다.
- Runs는 몇 개의 테스트를 실행했는가?, Errors는 몇 개의 테스트에서 에러가 났는가?, Faliures는 몇 개의 테스트를 실패했는지 나온다.

<br>

#### Assert Method(단정 메소드)
- @Test에서 사용되는 메소드이며 테스트 케이스의 수행 결과를 판별합니다.
- assertEquals(x, y) : 객체 x(예상값), 객체 y(실제값) 이 같으면 테스트가 통과됩니다. 
> - 매개변수 하나를 추가해주면 오차범위를 정할 수 있습니다.
- assertArrayEquals(x, y) : 배열 x와 배열 y가 같으면 테스트가 통과됩니다.
- assertTrue(x) : 리턴값이 true이면 테스트가 통과됩니다.
> - assertTrue(message, condition) : condition이 true면 message를 출력합니다.
- assertFalse(x) : 리턴값이 false이면 테스트가 통과합니다.
- assertNull(x) : 객체 x가 null이면 테스트가 통과됩니다.
- assertNotNull(x) : 객체 x가 null이 아니면 테스트가 통과됩니다.
- assertSame(x, y) : 객체 x와 객체 y가 같은 객체를 참조하고 있으면 테스트가 통과됩니다.
> - assertEquals(x, y)는 두 객체의 두 값을 비교합니다.
- assertNotSame() : 객체 x와  객체 y가 같은 객체를 참고하고 있지 않으면 테스트가 통과됩니다.
- assertThat(x, y) :  x에는 비교 대상의 실제값(T actual), y에는 비교 로직이 담긴 Matcher( Matcher<? super T> matcher)가 사용됩니다.
> 예시 :
>
>```java
>	@Test
>    public void testAdd() {
>        int result = 2+10;
>        assertThat(result, is(12));
>    }
>```  
> 
> - JUnit의 Matcher를 정리해주신 블로그입니다. 
> > - [JUnit의 assertThat_Beom Dev Log](https://beomseok95.tistory.com/250) 

<br>

#### AssertJ
- JUnit에서 사용되면서 메소드 체이닝을 지원하기 때문에 더 직관적인 테스트 코드 작성을 도와주는 라이브러리이다.
- Java8 이상은 3.x 버전, Java7 이하는 2.x 버전을 사용하면 된다. 
- 모든 테스트 코드는 assertThat()에서 시작됩니다.
- `assertThat(테스트 타겟).메소드1.메소드2.,,,,` 

```java
import static org.assertj.core.api.Assertions.*;

assetThat("타겟 문자열")
			.contins("a") //a 문자열을 포함하고
			.doseNotContain("b") //b 문자열을 포함하지 않고
            .isNotEmpty() //비어있지 않고
            .startsWith("c") //c 문자열로 시작하고
            .endWith("d") //d 문자열로 끝나고
            .isEqualTo("e") //e 문자열과 같고
```


```java
assetThat("타겟 숫자")
			.isGreaterThan(0) // 0보다 크고
            .isLessThan(1) // 1보다 작고
            .isPositive() // 양수이고
            .isNegative() // 음수이고
            .isEqualTo(2) // 2와 같고
            .isEqualTo(3, offset(0.1d)) //오프셋 0.1기준으로 값이 같고
```

- 더 많은 메소드는 공식 문서를 통해 알 수 있다.
- [Assertj 공식 사이트](http://joel-costigliola.github.io/assertj/index.html)

<br>

### REST API

- 과거에 올렸던 REST API, Web API의 특징, URI의 특징, 상태 코드에 대하여 정리한 포스트이다. + 수정할 점이 많아서 보충했다.
- [REST API에 관하여](https://ddungi.github.io/web/2022/03/15/restapi/)
- 아래부터는 Spring Boot에서 사용되는 요청매핑 Annotation에 대해 정리해보려 한다.

<br>

#### @RequestMapping
- 해당 Annotation은 클래스와 메소드 모두 앞에 붙일 수 있다.
- 전에는 모든 메소드 단에 붙여서 모든 요청을 해당 Annotation으로 받았다.
- `@RequestMapping(value = "/getStudent.do", method = RequestMethod.GET)`을 이용하여 Get, Post 방식을 나눠서 받기도 했다.
> - 위 방식은 value값을 URI가 정보의 자원을 효율적으로 보여주기에 어렵고, RESTful 하다고 할 수 없다.
- 그래서 클래스 단에 Annotation을 붙여 공통된 URI를 지정할 때 사용한다.

- 아래 Annotation을 사용하면 RESTful한 요청을 받을 수 있다.

##### @GetMapping
- SQL문에서 SELECT, 조회할 때 사용된다. 
- 파라미터 정보가 URL에 담겨 오기 때문에 보안성이 좋지 않다.

##### @PostMapping
- SQL문에서 INSERT, 데이터를 저장할 때 사용된다.
- 파라미터 정보가 HTTP 바디에 담겨 오기 때문에 보안성이 좋다.

##### @PutMapping
- SQL문에서 UPDATE, 데이터를 수정할 때 사용된다.

##### @DeleteMapping
- SQL문에서 DELETE, 데이터를 삭제할 때 사용된다. 

<br>


```java
// import 생략
@RequestMapping("/school")
@RestController
public class RESTController {
		
        @Autowired
		StudentService studentService;
        
		// GET : SELECT
        @GetMapping("/student")
		public String getStudent(@RequestBody Student student) { 
			Student student = studentService.getStudent()
			return "조회한 학생 정보 : "+ student.toString();
		}

		// POST : INSERT
		@PostMapping("/student")
		public String postStudent(@RequestBody Student student) { 				studentService.insertStudent();
			return "등록된 학생 정보 :  " +student.toString();
		}
		
		// PUT : UPDATE
		@PutMapping("/student")
		public String putStudent(@RequestBody Student student) {
			studentService.updateStudent();
            return "학생 정보 수정 처리";
		}
		
		// DELETE : DELETE
		@DeleteMapping("/student")
		public String deleteStudent(@RequestParam int seq) {
        	studentService.deleteStudent();
			return seq + "번 학생 정보 삭제";
		}
}
```

<br>

### 추가적인 Annotation
- 일반적으로 클라이언트와 서버 간 데이터 수송신을 할 때 JSON이라는 형태로 주고 받는다.
- @ResponseBody : 메소드 위에 붙으며, 리턴하는 Java 객체를 JSON 로  변환해서 HTTP 응답 바디에 넣어 화면에 출력할 수 있다.

- @RestController : 
> - 클래스 위에 붙으며, 클래스 내 모든 메소드의 리턴값을 바디에 담아 응답한다.
> - 매 메소드마다 @ResponseBody를 붙이기 번거로울 때 이용할 수 있다. 
- @Controller : 클래스 위에 붙으며, 해당 클래스는 Controller의 역할을 한다는 의미이고 기본적으로 return 값과 viewResolver를 이용해 이동할 View를 찾는다. 
- @RequestBody : JSON로 들어온 요청을 Java 객체로 변환하여 데이터를 받는다.
- @RequestParam : 전달받은 파라미터 하나를 매핑해준다.
- @PathVarible : URI 뒤에 받은 파라미터를 매개변수에 매핑해준다.
> - 프론트 단에서 id 속성으로 정한 값은 Server 단으로 넘어오지 않아서 name 속성만 받을 수 있다. 
> > - id : JavaScript에서 사용하기 위해 지정한다.
> > - name : 서버로 파라미터 전송을 하기 위해 지정한다.

```java
@GetMapping("/school/class/{number}")
public Class getClass(@PathVariable int number){
		//생략
}
```

<br>

Reference:
- [inflean_img](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC_renew)
- [org.junit Class Assert](http://junit.sourceforge.net/javadoc/org/junit/Assert.html)
- [Assertj 공식 사이트](http://joel-costigliola.github.io/assertj/index.html)
- [AssertJ 소개_DaleSeo](https://www.daleseo.com/assertj/)
- [JUnit의 assertThat보다 assertj의 assertThat을 써야하는 이유_JW공부스토리](https://jwkim96.tistory.com/168)
- [[HTML] name과 id의 차이점_ 지혜](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=bluegriffin&logNo=40132483542)