---
layout: post
title: '[JAVA, SPRINGBOOT] JUNIT, REST API'
subtitle: 'Test case, @GetMappping, @PostMapping, @PutMapping, @DeleteMapping'
date: 2022-07-08 12:00:00 +0900
categories: 'JAVA'
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

- src/main/test/패키지에서 New-JUnit Test Case를 선택한다.
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
- assertNotSame(x, y) : 객체 x와  객체 y가 같은 객체를 참고하고 있지 않으면 테스트가 통과됩니다.
- assertThat
https://jwkim96.tistory.com/168
http://junit.sourceforge.net/javadoc/org/junit/Assert.html
<br>
### REST API



<br>

<br>
Reference:
- [inflean_img](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC_renew)
- [org.junit Class Assert](http://junit.sourceforge.net/javadoc/org/junit/Assert.html)
