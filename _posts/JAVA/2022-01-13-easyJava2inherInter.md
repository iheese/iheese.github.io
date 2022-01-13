---
layout: post
title: '쉽게 배우는 자바2'
subtitle: '상속(Inheritance), 인터페이스(Interface)'
date: 2022-01-13 17:00:00 +0900
categories: 'JAVA'
---

## 상속(Inheritance)
```java
class Cal {
	public int sum(int v1, int v2) {
		return v1 + v2;  //부모 클래스 더하기 구현
	}
}
class Cal3 extends Cal{
	   //Cal에서 상속된 자식  Cal3; extends 사용
}
...
public class InheritanceApp {

	public static void main(String[] args) {
		Cal c = new Cal();
		System.out.println(c.sum(2, 1)); 
        
		Cal3 c3 = new Cal3();
		System.out.println(c3.sum(2, 1));
 //Cal3은 아무 기능 없지만 Cal에게 기능을 상속 받아 더하기 메소드를 사용한다. 
	}
}
```

- 상속은 한 클래스당 한 개만 받을 수 있다.

<br>

#### 재정의 (Overriding)

```java
class Cal {
	public int sum(int v1, int v2) {
		return v1 + v2;
	}
}
class Cal3 extends Cal{
	public int minus(int v1, int v2) {
		return v1 - v2;   //이런 식의 상속 구조에서
	}
}
```
```java
class Cal3 extends Cal{
	
	public int sum(int v1, int v2) {
    	System.out.println("Cal3!!!");
		return v1 + v2;   
	}
 //부모 클래스 기능에 더해 Cal3!!!을 출력하는 기능까지 더하는 것
	public int minus(int v1, int v2) {
		return v1 - v2;
	}
}
```

- eclipse의 Source - Override/Implement Methods 로도 사용 가능

<br>

### 오버로딩(Overloading)
```java
class Cal {
	public int sum(int v1, int v2) {
		return v1 + v2;
	}
	// Overloading
	public int sum(int v1, int v2, int v3) {
		return v1 + v2 + v3;
	}
}

public class InheritanceApp {

	public static void main(String[] args) {
		Cal c = new Cal();
		System.out.println(c.sum(2, 1)); //3
        System.out.println(c.sum(2, 1, 1));  //4
        }
  }
```

- 같은 이름의 메소드지만 파라미터 형식을 보고 자바가 택해서 값을 호출한다.
- 상속과는 관련이 없으므로 부모 클래스에 있던 메소드를 자식 클래스로 옮겨도 된다. 
<br>

#### this, super
```java
class Cal {
	public int sum(int v1, int v2) {
		return v1 + v2; 
	}
	// Overloading
	public int sum(int v1, int v2, int v3) {
		return this.sum(v1, v2) + v3;
	} //생성자 값을 받을 때 인스턴스의 값을 this로 가져온다. 
}
class Cal3 extends Cal{
	// Overriding
	public int sum(int v1, int v2) {
		System.out.println("Cal3!!!");
		return super.sum(v1, v2);
	}  //자식 클래스에서 부모 클래스의 메소드와 변수에 접근 가능

	public int minus(int v1, int v2) {
		return v1 - v2;
	}
}
```
<br>
### 생성자와 클래스
```java
class Cal{
    int v1,v2;
    Cal(int v1, int v2){
        System.out.println("Cal init!!");
        this.v1 = v1; this.v2 = v2;
    }
    public int sum(){return this.v1+v2;}
}
class Cal3 extends Cal{
    Cal3(int v1, int v2) {
        super(v1, v2);
        System.out.println("Cal3 init!!");
    }
    public int minus(){return this.v1-v2;}
}
```

- 부모 클래스가 생성자가 있을 때 자식 클래스에 생성자를 만들지 않으면 컴파일이 불가능하다. 
- 생성자를 만들 때는 클래스의 이름으로 메소드를 만드는데 부모 클래스에서 생성자를 만들었다고 자식 클래스의 생성자를 자동으로 가져올 수는 없다. (이름 중복 불가)

<br>

## 인터페이스(Interface)
- 보통 대문자로 시작, '~할 수 있는' 뜻을 가진 형용사 이름을 붙임 
- 인터페이스는 여러 개 적용할 수 있다. 
- 어떤 클래스를 구성할 때 꼭 필요한 약속, 규제 같은 것

#### 다형성(polymorphism)
```java
interface Calculable { //인터페이스 구현
	double PI = 3.14;   //변수에는 값이 들어간다. 
	int sum(int v1, int v2);  클래스에 대한 제한,약속
}   //v1,v2 라는 정수를 받고 sum이라는 메소드로 정수 출력
interface Printable {
	void print();  //print라는 메소드
}
class RealCal implements Calculable, Printable { 
//인터페이스가 적용된 클래스 

	public int sum(int v1, int v2) {
		return v1 + v2;
	}

	public void print() {
		System.out.println("this is RealCal!!!");
	}	
	
}

public class InterfaceApp {

	public static void main(String[] args) {
		//Calculable c = new RealCal();
        //위와 같이 정의되면 계산 메소드, PI 만 사용가능
        //Printable c = new RealCal();
        //위와 같이 정의되면 출력 메소드만 사용가능
		System.out.println(c.sum(2, 1));
		c.print(); // Compile Error
		System.out.println(c.PI);
	}

}
```

- 객체의 타입이 부모 클래스, 인터페이스, 자식 클래스 등으로 다양하게 적용이 가능한데 인스턴스로 만든 객체(일반적으로 만드는 방식;자식 클래스의 인스턴스) 처럼 행동하는 것을 '다형성' 이라 한다. 

<br>

Reference:
 - 쉽게 배우는 자바2_생활코딩, 부스트코스를 통해 공부한 내용입니다. 
