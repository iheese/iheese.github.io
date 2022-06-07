---
layout: post
title: '[JAVA, SPRING] DI(Dependeny Injection), IoC (Inversion of Control)'
subtitle: '의존성 주입, 제어의 역전, xml파일, Bean, Annotation'
date: 2022-06-03 12:00:00 +0900
categories: 'JAVA'
background: '/img/posts/etc/spring.jpg'
---

### Dependeny (의존성)

```java
class A{
	private B b;
    
    public A(){
    	this.b = new B();
    }
}
```

- A는 B에 의존한다. 
- A가 생성될 때 B가 먼저 생성되어야 하며, B가 변하면 A도 변하게 된다.
- 위 같은 코드는 재활용성이 떨어지면서 결합도가 높아지게 된다.
- 일반적으로 결합도가 높은 코드는 유지보수 측면에서 좋지 않다. 

<br>

### DI (Dependeny Injection, 의존성 주입)

- 결합도가 낮은 코드를 위해서는 주입받는 객체를 인터페이스로 만들어 추상화시키면 된다. 
- 이 같은 방법은 인터페이스 구현체들을 통해 재활용성을 높여 반복을 줄일 수 있고 결합도가 낮아지는 효과를 가져온다.
- DI 구현 방식에는 Constructor(생성자), Setter(수정자)를 이용하는 방법이 있다.  


B 인터페이스

```java
interface B{
	void methodB();
}
```

B 인터페이스 구현체

```java
public class BB implements B{
	@Override
	public void methodB(){
    	System.out.println("I'm BB from B");
    }
}

------------------------------------------------------------

public class BC implements B{
	@Override
	public void methodB(){
    	System.out.println("I'm BC from B");
    }
}
```

- Constructor(생성자)를 이용한 방법

package.A

```java
class A{
	private B b;
    
    public A(B b){
    	this.b = b;
    }
    public void methodA() {
    	b.methodB();
    }
}
```

Main 클래스

```java
public class Main {
    public static void main(String[] args) {
    A a1 = new A(new BB());
    A a2 = new A(new BC());
    
    a1.methodA(); //I'm BB from B
    a2.methodA(); //I'm BC from B
	}
}
```

<br>

- Setter(수정자)를 이용한 방법

package.A

```java
class A{
	private  B b;
    
    public void setB(B b){
    	this.b = b;
    }
    
    public void methodA() {
    	b.methodB();
    }
}
```

Main 클래스

```java
public class Main {
    public static void main(String[] args) {
    A a = new A();
    
    a.setB(new BB());
    a.methodA(); //I'm BB from B
    a.setB(new BC());
    a.methodA(); //I'm BC from B
	}
}
```

- Constructor 방법을  사용하면 객체 주입없이 의존적인 객체를 생성할 수 없고 null값을 넣지 않는 이상 NullPointerException이 발생하지 않는다.
-  Constructor 방법은 의존관계를 직관적으로 볼 수 있어 좋다.
- 또한 Constructor 방법은 주입객체(B)를 final(선언되면서 초기화되어야 함)로 선언하여 주입받는 객체 내부 변경을 막을 수 있다. 
- 하지만 Constructor 방법은 생성자가 매개변수를 가지고 있을 때는 모든 경우의 수를 고려해 생성자를 만들어야 하기 때문에 코드의 길이가 늘어나는 단점이 있다. 

<br>

### Spring에서 DI(Dependeny Injection)

- Servlet에서 Servlet Container가 Servlet 객체의 Lifecycle을 관리했던 것처럼 Spring에서도 Spring Container가 bean이라는 객체를 관리한다.
- 객체의 Lifecycle과 의존성 주입까지 처리해준다. 
- applicationContext.xml (파일명 무관) 라는 xml파일(Spring Config file)을 통해 bean을 관리할 수 있다. 
> - Spring 파일 생성 - Spring Bean Configuration file을 통해 생성이 가능하다.
> - applicationContext.xml 파일을 Namespaces로 보면 자신이 사용할 태그를 추가할 수 있다.(Eclipse 기준)


- constructor(생성자) 를 이용한 방법

package.A

```java
class A{
	private  B b;
    
    public void A(B b){
    	this.b = b;
    }
    public void methodA() {
    	b.methodB();
    }
 }
```

applicationContext.xml 파일

```xml
<beans> <!--생략 -->

<bean id="a" class="package.A">
	<constructor-arg ref="bc"></constructor-arg>
    <!-- <constructor-arg value="10"></constructor-arg>
    A를 생성할 때 int 인자를 주입할 때는 value 속성 사용-->
</bean>

<bean id="bc" class="package.BC"></bean>

</beans>
```

<BR>

- Setter(수정자)를 이용한 방법

package.A

```java
class A{
	private  B b;
    
    public void setB(B b){
    	this.b = b;
    }
    public void methodA() {
    	b.methodB();
    }
}
```

applicationContext.xml 파일

```xml
<beans> <!--생략 -->

<bean id="a" class="package.A">
	<property name="b" ref="bc"></property>
    <!--<property name="c" value="10"></property>
 A 생성할 때 int 인자를 주입할 때는 value 속성 사용-->
</bean>

<bean id="bc" class="package.BC"></bean>

</beans>
```

Main 클래스

```java
import org.springframework.context.support.GenericXmlApplicationContext;
public class Main {
	public static void main(String[] args) {
GenericXmlApplicationContext container=
		new GenericXmlApplicationContext("applicationContext.xml");
        
 	A a= (A) container.getBean("a"); 
     a.methodA(); // I'm BC from B
    //getBean은 Object 타입을 반환하기 때문에 명시적 형 변환이 필요하다. 
    // getBean을 이용한 방식은  Spring Container로부터 객체를 획득하는 것이다.(Lookup) 
    
    }
 }
```

- bean 등록시 id 속성을 생략하면 `package.A#01` 형식으로 임의로 배정된다. 
- class 속성은 생략할 수 없다. 
- Spring container는 기본적으로 pre-loading(컨테이너 생성시 객체가 생성)이다. `lazy-init="true"` 를 bean 속성에 추가하면 lazy-loading(요청시 객체 생성)으로 변경할 수 있다.
- Spring container는 기본적으로 singleton(하나의 객체만 생성) 형식을 띄고 있으며 `scope="prototype"` 으로 요청할 때마다 새로운 객체를 생성하게 할 수 있다. 
- `init-method="method이름"`, `destroy-method="method이름"` 를 이용해 객체 생성 후, 소멸 전 메소드를 실행할 수 있다. 

<br>

### IoC (Inversion of Control, 제어의 역전)

- IoC는 객체의 흐름, Lifecycle 등의 제어를 제 3자에게 위임하는 프로그래밍 모델이다. (템플릿 메소드 패턴에서도 찾을 수 있다.)
- 위의 예시에서는 Spring container가 xml 파일을  통해 어플리케이션 운용에 필요한 객체의 생성과 의존 관계를 처리하는 것을 IoC라 한다.
- 모든 객체를 xml 파일로 관리하는 것은 xml 파일의 복잡도를 높여 유지 보수에 좋지 않다. 

<br>

### Component-Scan , Annotation

- 먼저 Namespaces에서 context 사용을 체크해준다.

applicationContext.xml 파일

```xml
<beans> <!--생략 -->
	<context:component-scan base-package="package.~"></context:component-scan>
<!--base-package 하위 패키지에서 Annotation을 찾는다. -->
</bean>
</beans>
```

package.A

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component("a") //id="a"로 하는 bean으로 등록하는 것과 같다.
class A{
	@Autowired //해당 타입의 객체를 찾아서 자동으로 할당해준다.
	private B b;
    
    public void setB(B b) {
    	this.b=b;
    }
    
    public void methodA() {
    	b.methodB();
    }
}
```

B 인터페이스 구현체인 BC 클래스

```java
import org.springframework.stereotype.Component;

@Component  // @Autowired가 연결될 객체
public class BC implements B{
	public void methodB(){
    	System.out.println("I'm BC from B");
    }
}
```

Main 클래스

```java
import org.springframework.context.support.GenericXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
    
    	GenericXmlApplicationContext container=
    			new GenericXmlApplicationContext("applicationContext2.xml");
    	
    	A a=(A)container.getBean("a");
    	a.methodA(); //I'm BC from B
    
	}
}
```

- @Component는 xml 파일에서 bean 등록하는 것과 같다. 
> - 해당 클래스를 메모리에 올려 접근이 가능한 형태로 만들어준다.
> - @Component("id") 로 id를 지정해줄 수 있다. 
- @Autowired는 Type Injection이라고 하기도 하며, 변수 위에 설정한다.
> - 메모리 내 해당 타입의 객체를 찾아 자동으로 할당해준다.
> - 해당 타입의 객체는 메모리에 존재해야 하고 유니크하게 존재해야 한다. 
- 메모리에 한 타입의 객체가 두 개 있을 때 @Autowired, @Qulifier("id")을 함께 적어 id 이름으로 객체를 할당 받을 수 있다.
- @Resource(name="id")로 id 이름으로 객체를 할당 받을 수도 있다.

<br>

#### Layer별 Annotation
- @Component를 상속하고 있으며 객체의 구체적인 클래스 처리와 명시적인 연관성을 부여할 수 있다. 
- @Service : 비즈니스 로직을 처리하는 Service 클래스, Service 인터페이스의 구현체에 주로 붙는다.
- @Repository : DB 연동을 처리하는 DAO 클래스, DAO 클래스에 주로 붙는다.
- @Controller : 사용자 요청을 제어하는 Controller 클래스, Controller 클래스에 주로 붙는다. 

<br>

Reference:
- [inflean_img](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC_renew)
- [의존관계 주입(Dependency Injection) 쉽게 이해하기_ Tecoble](https://tecoble.techcourse.co.kr/post/2021-04-27-dependency-injection/)
- [IoC, DI란 무엇일까_백문이불여일견](https://biggwang.github.io/2019/08/31/Spring/IoC,%20DI%EB%9E%80%20%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C/)
- [탬플릿 메소드 패턴과 제어역전(IoC)_happygrammer](https://happygrammer.tistory.com/65)
