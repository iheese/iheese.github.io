---
layout: post
title: "[JAVA] INHERITANCE, OVERRIDING, FINAL, TYPE CONVERSION"
subtitle: "extends, super, this, final, Implicit Type Conversion(묵시적 형 변환), Explicit Type Conversion(명시적 형 변환) "
date: 2022-04-13 12:00:00 +0900
categories: "JAVA"
background: "/img/posts/javaetc/java.png"
---

## Inheritance(상속)
-  A 클래스가 B 클래스에 상속되었다.
> `class B extends A{ }` : java에서 사용법
> - A : 부모 클래스 , B : 자식 클래스
- 자식 클래스는 부모 클래스보다 추가적인 메소드, 멤버변수를 가지고 있다.
- 부모 클래스가 상속되면 자식 클래스에서 따로 선언하지 않아도 접근이 가능하다.
- 상속을 이용하면 코드 반복을 줄여 메모리 낭비를 줄일 수 있다. 
> - public : 어디에서든 항상 접근 가능
> - protected : 같은 패키지와 상속 관계에서 접근 가능
> - default : 같은 패키지에서만 접근 가능, 상속 관계도 같은 패키지에 있으면 접근 가능
> - private: 같은 클래스 내부에서만 접근 가능

Ex) 간단한 예제로 보는 상속 관계


- 부모 클래스에서는 이름과 나이를 설정하고 출력하는 메소드가 있다.

```java
public class Parent{
	String name;
	int age;
	
	public void setName(String name) {
		this.name = name;
	}
	public void setAge(int age) {
		this.age = age;
	}
    public void showNameAge() {
    	System.out.println("이름은 "+ name + ", 나이는 " + age +" 입니다." );
    }
}

```

<br>
- 자식 클래스는 부모 클래스를 상속받았다.
- 그 외에 자식 클래스는 직업과 키를 설정하고 출력하는 메소드가 있다. 

```java
public class Child extends Parent{
	String job;
	int height;
	
	public void setJob(String job) {
		this.job = job;
	}
	
	public void setHeight(int height) {
		this.height = height;
	}
	
	public void showJobHeight() {
    	System.out.println("직업은 "+ job + ", 키는 " + height +" 입니다." );
    }
}

```

<br>
- 부모 인스턴스는 자신이 가진 메소드를 당연히 사용한다.
- 자식 클래스는 부모 클래스를 상속받았기 때문에 부모 클래스의 메소드(이름과 나이 설정 및 출력)를 사용할 수 있다.
- 자식 클래스는 부모 클래스의 메소드 외에 자신의 메소드 역시 사용할 수 있다. 

```java
public class ChildTest {
	public static void main(String[] args) {
	    Parent parent = new Parent();
	    parent.setName("부모");
	    parent.setAge(50);
	    parent.showNameAge(); //자신의 메소드이므로  문제될 것이 없다. 
	     //이름은 부모, 나이는 50 입니다.
	    
	    Child child = new Child();
	    child.setName("자식");
	    child.setAge(22);
	    child.showNameAge();
	    //이름은 자식, 나이는 22 입니다.
	    child.setJob("요리사");
	    child.setHeight(185);
	    child.showJobHeight();
	    //직업은 요리사, 키는 185 입니다.
	}	   
}
```

<br>

## Overriding (오버라이딩)
- 변수 오버라이딩 : 부모 클래스가 자식 클래스에 상속되었을 때 부모 클래스의 멤버 변수 이름이 자식 클래스의 멤버 변수 이름과 같다면 부모 클래스의 멤버 변수는 상속되지 않는다. (자식 클래스 멤버 변수로 덮어쓰워진다.)

```java
public class Parent{
	String name;
    int age;
}
```

```java
public class Child extends Parent{
    String job;
    double age;
}
```

- 부모 클래스에서는 int형으로 age 변수를 사용했지만 자식 클래스에서는 double형으로 age 변수를 사용하게 된다.

<br>
- 메소드 오버라이딩 : 부모 클래스가 자식 클래스에 상속되었을 때 부모 클래스 메소드의 시그니처(이름, 매개변수, 리턴타입)가 자식 클래스의 메소드와 같다면 부모 클래스의 메소드는 상속되지 않는다.

```java
public class Parent{
	String name;
    int age;
    
    public void showInfo(){
    	System.out.println(name + "입니다. 나이는 " + age + "살입니다.");
    }
}
```

```java
public class Child extends Parent{
    String job;
    
    public void showInfo(){
    	System.out.println(name + "입니다. 나이는 " + age + "살이고" + "제 직업은 "+ job +" 입니다.");
    }
}
```

- 자식 클래스의 메소드와 부모 클래스의 메소드는 같은 이름을 가져야 한다.
- 부모 클래스의 메소드 반환 타입이 void면 자식 클래스의 메소드 반환 타입이 void여야 한다.
- 부모 클래스의 메소드가 타입 반환이 존재하면 자식 클래스 메소드의 반환 타입은 변경 가능하다.
- 자식 클래스의 메소드는 부모 클래스의 메소드보다 더 좁은 접근 제어자의 범위로 변경할 수 없다.
- 부모 클래스의 메소드보다 더 큰 범위의 예외로 선언할 수 없다. 

<BR>

### Overloading(오버로딩) vs Overriding(오버라이딩)
- 오버 로딩 : 메소드를 중복으로 정의한다. 객체에 접근하기 용이하게 하기 위함이다.
- 오버 라이딩 : 상속받은 기존의 메소드를 재정의하는 것이다. 코드 길이를 줄인다.

```java
public class Parent{
	String name;
    int age;
    
    public void showInfo(){
    	System.out.println(name + "입니다. 나이는 " + age + "살입니다.");
    }
}
```

```java
public class Child extends Parent{
    String job;
    
    public void showInfo(){
    	System.out.println(name + "입니다. 나이는 " + age + "살이고" + "제 직업은 "+ job +" 입니다."); //오버라이딩
    }
    
    public void showInfo(String job){
    	System.out.println(name + "," + age + "," + job); //오버로딩
    }
}
```

```java
public class ChildTest {
	public static void main(String[] args) {
  		Child child = new Child();
        
        child.showInfo();
        //null입니다. 나이는 0살이고제 직업은 null 입니다.
        child.showInfo("소방관"); //오버로딩 사용
        //null,0,소방관
  	}
  }
```

<br>

## This, Super, Final
- ##### this
> - 클래스의 멤버 변수 이름과 메소드/생성자의 지역 변수 이름이 같을 때 사용한다.

```java
public class Parent{
	String name;
    int age;
    
    public Parent(String name, int age){
    	this.name=name; 
        this.age=age;
       //이 생성자의 지역 변수가 클래스의 멤버 변수와 같다는 것을 의미한다. 
    }
}
```

> - 클래스에 오버로딩된 다른 생성자 호출할 때 사용한다. 

```java
public class Parent{
	String name;
    int age;
    
    public Parent(String name){
    	this(name, 0); //오버로딩된 다른 생성자를 호출했다. 
    }
    
    public Parent(String name, int age){
    	this.name=name; 
        this.age=age;
    }
}
```

> - 객체 자신의 참조값을 전달할 때 사용한다. 

```java
public class Parent{
	String name;
    int age;
       
    public Parent(String name, int age){
    	this.name=name; 
        this.age=age;
    }    
    public Parent getParentInstance(){
    	return this; //Parent 객체의 참조값을 리턴했다.
    }
}
```

<br>
- ##### super
> - 상속받은 부모 클래스의 멤버변수나 메소드를 참조하는 변수이다.

```java
public class Parent{
	String name="lee";
}
```

```java
public class Child extends Parent{
    String name; //변수 오버라이딩
    
    public void showInfo(){
    	
        System.out.println(name);
    	System.out.println(this.name);
    	System.out.println(super.name);
    }
}
```

```java

public class ChildTest {
	public static void main(String[] args) {
  		Child child = new Child();
        child.name = "kim";
        child.showInfo();
        // kim : 같은 클래스의 값을 받아왔다.
        // kim : 같은 클래스의 멤버변수 값을 받아왔다. 
        // lee : 자식 클래스에서 부모 클래스의 변수값을 받아왔다.
  	}
}
```


- ##### super()
> - 부모 클래스의 생성자를 호출할 때 사용된다. 
> - 일반적으로 자식 클래스의 객체를 생성하면 부모 클래스의 생성자가 먼저 실행된 뒤 순차적으로 실행된다.(부모 클래스의 멤버변수가 초기화되어야 자식 클래스에서 상속받은 변수를 사용할 수 있다.) 
> - 보통 부모 클래스에 기본 생성자가 있다면 자식 클래스에서 자동으로 super()를 호출해 초기화해준다.
> - 그러나 부모 클래스에 기본 생성자가 없다면, 반드시 super()로 생성자를 명시해주어야 한다.

```java
public class Parent{
	String name;
    
    Parent(String name){
    	System.out.println(name + " 매개변수와 부모 생성자 생성");
    }
}
```

```java
public class Child extends Parent{
        
   	Child(){
    	super("lee");
        System.out.println("자식 생성자 생성");
    	}
    }
```

```java
public class ChildTest {
	public static void main(String[] args) {
  		Child child = new Child();
        //lee 매개변수와 부모 생성자 생성
		//자식 생성자 생성
  	}
  }
```

- 매개변수를 가지는 생성자 생성시 기본 생성자를 명시적으로 선언해주는 것이 좋다. 

<br>
- ##### final
> - 변경할 수 없다는 의미를 가지고 있다.
> - final + 변수 : 값 변경이 불가한 상수가 된다.(재할당이 불가하다.)
> - final + 메소드 : 오버라이딩이 불가하다.
> - final + 클래스 : 상속이 불가하다.

```java
final class Parent{ //final 클래스 생성
	final String name="lee"; //final 변수 생성, 상수화
    
    final void showInfo() {
    	System.out.println("이름은 "+name+ "입니다.");
    } //final 메소드 생성
}
```

```java
//public class Child extends Parent{ //상속 불가

		  // String name= "kim"; //재할당 불가
        
   		//public void showInfo() {
   			System.out.println(name + "입니다. 오버라이딩 불가");
   		} //오버라이딩 불가
    }
```

<br>

## Type conversion (묵시적 형 변환, 명시적 형 변환) in 상속

- 묵시적 형 변환 : 자식 객체를 부모 타입의 참조 변수에 자동적으로 할당하는 것을 의미한다. 
> - 할당은 되지만 부모 타입의 참조 변수에서는 부모 변수에서만 접근이 가능하다.
- 명시적 형 변환 : 부모 객체를 자식 타입의 참조 변수에 강제적으로 할당하는 것을 의미한다. 

```java
public class GrandParent{
	int a = 1;  // GrandParent 클래스는 a 멤버변수만 가지고 있다.
}
```

```java
public class Parent extends GrandParent{
	int b = 2; // Perent 클래스는 a(상속), b 맴버변수를 가지고 있다.
}
```

```java
public class Son extends Parent{
	int c = 3; // Son 클래스는 a(상속), b(상속), c 멤버변수를 가지고 있다. 
}
```

```java
public class Family{
	public static void main(String[] args){
    	GrandParent park = new GrandParent(); //일반적인 객체 생성
        GrandParent lee = new Parent(); 
        // a, b 멤버변수를 가진 Parent가 GrandParent에 할당될 수 있다.
        GrandParent cho = new Son();
        // a, b, c 멤버변수를 가진 Son이 GrandParent에 할당될 수 있다. 
        System.out.print(park.a); //1
        System.out.print(park.b); //오류
        System.out.print(park.c); //오류
        System.out.print(lee.a); //1
        System.out.print(lee.b); //오류
        System.out.print(lee.c); //오류
        System.out.print(cho.a); //1
        System.out.print(cho.b); //오류
        System.out.print(cho.c); //오류
        //그러나 위 세 경우 모두 a에만 접근이 가능하다. 
    }
}

```

- 위의 예시처럼 자식 객체가 부모 참조 변수에 할당되는 것을 묵시적 형 변환이라 한다. 
- 부모 타입에 해당되는 범위만 접근이 가능하다.
- 형제 단계(같은 상속 단계) 에서는 묵시적 타입 변환이 불가능하다.
- 부모 객체를 자식 타입 참조 변수에 할당하면 선언된 데이터보다 작은 데이터가 들어와 데이터가 유실되므로 불가하다. 

Ex) 명시적 형 변환 에시

```java
public class Family{
	public static void main(String[] args){
    	GrandParent lee = new Son(); //묵시적 형 변환
        Son lee2 =(Son) lee; //명시적 형 변환
        }
    }
```

- 한 번 묵시적 형 변환했던 객체를 자신의 타입으로 되돌리는 형태만 가능하다. 

### 묵시적 형 변환이 중요한 이유
- 최상위 클래스를 통해 상속된 수많은 클래스의 객체를 생성할 수 있게 된다.
- 수정사항이 필요할 경우 최상위 클래스 내부만 수정하면 된다.
- 즉, 공통된 최상위 객체를 통해 유지보수가 쉬워진다. 

<br>

Reference:
- 백엔드 개발 필수_패스트캠퍼스를 학습하고 정리한 내용입니다. 
- [java img_크레벅스](https://www.crebugs.com/product/view.php?idx=7382&code=1412)  
- [메소드 오버라이딩_TCP](http://www.tcpschool.com/java/java_inheritance_overriding)
- [fianl제어자_TCP](http://www.tcpschool.com/java/java_modifier_ectModifier)