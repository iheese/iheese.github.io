---
layout: post
title: "INSTANCE, CONSTRUCTOR IN JAVA"
subtitle: "object, instance, heap, constructor"
date: 2022-04-06 12:00:00 +0900
categories: "JAVA"
---

## Object (객체)
- 구체적, 추상적 데이터의 단위, 의사나 행위가 미치는 대상

## OOP (Object Oriented Programming, 객체 지향 프로그래밍)
- 객체를 정의한다.
- 객체가 제공하는 기능을 구현한다.
- 각 객체가 제공하는 기능 간의 소통을 통해 협력을 구현한다. 

<br>
## Class (클래스)
- 객체를 코드로 나타낸 것을 의미한다. 
- 예시

```java
public class Person{
	int age;
    String name;
    char sex;
    int personHeight;
    int personWeight;  //멤버 변수 : 객체 내 명사적 역할
    
    Person(){} //생성자 메소드: 초기화 역할
    
    public void printInfo(){
    	System.out.println(name + "님은 " + age + "살 입니다.");
    }
    public void printBodyInfo(){
    	System.out.println(name + "님의 " + "몸무게는 " + personWeight + "kg, 키는 " + personHeight + "cm 입니다.");
    }  //일반 메소드 : 객체 내 동사적 역할
}
````

> - 클래스의 구성
> > - 멤버 변수 
> > > - 객체의 명사적 기능, 객체 내에 사용되는 변수, 객체 속성
> > > - 객체에 유일하게 저장되는 요소, 생성자 메소드와 일반 메소드는 저장되지 않는다.
> > - 생성자 메소드
> > > - 멤버 변수의 초기화 역할
> > > - 클래스로부터 객체(인스턴스) 생성할 때 호출된다.
> > - 일반 메소드
> > > - 객체의 동사적 기능, 멤버 변수를 이용한 함수의 기능
> > > - 객체가 제공하는 기능적 요소

<br>
## instance(인스턴스)
- 클래스는 추상화된 객체이다. 객체에 다양한 값을 넣어 사용하기 위해 인스턴스가 필요하다.
- Ex) Person 이라는 클래스의 틀을 만들어놓고, 인스턴스를 통해 "Lee"라는 사람, "kim" 이라는 사람을 만들어 사용하는 것이다. 
- 인스턴스를 생성할 때는 new 연산자를 이용한다. 


```java

public class PersonTest {

	public static void main(String[] args) {
		
		Person personLee = new Person(); //인스턴스 생성
		personLee.name = "이길동";
		personLee.age = 25;   //멤버변수 설정
		
		
		personLee.printInfo(); //일반 메소드 호출
		
		Person personKim = new Person(); //인스턴스 생성
		personKim.name = "김장군";
		personKim.age = 55;   //멤버변수 설정
		
		
		personKim.printInfo(); //일반 메소드 호출
		
		System.out.println(personLee);
		System.out.println(personKim); 
        // 인스턴스를 출력하면 해당 메모리 주소가 나온다.
	}
}

```

- 생성된 인스턴스는 동적 메모리(heap memory)에 저장된다. 
- 자바에서는 사용하지 않는 동적 메모리를 주기적으로 수거한다.
- 각각의 인스턴스에는 각각의 동적 메모리의 주소가 저장된다.
- 즉 클래스를 기반으로 인스턴스가 생성된다. > heap 영역에 인스턴스의 멤버 변수 값들이 저장된다. > 인스턴스 자체에서는 heap 영역과 연결된 주소값을 저장된다. (이를 '참조변수' 라 한다. ) 

<br>

## constructor(생성자)
- 멤버 변수의 초기화를 위해 사용된다.
- 인스턴스를 생성하는 new 연산자와 사용될 때 호출된다. 
- 생성자는 클래스의 이름과 동일하게 구현하면 된다. (클래스 이름과 마찬가지로 보통 대문자 시작)
- void (X), 생성자는 리턴 타입이 없다.

```java

 	Person(){} //기본 생성자 메소드, 아무 매개변수를 가지고 있지 않다.

``` 

- 생성자를 생성하지 않으면 컴파일러가 기본 생성자 메소드를 생성해준다. 
- String: null, char: '\u0000' , int,byte,short: 0 ,long: 0L, double: 0.0, float: 0.0F, boolean: false, Stirng, 배열, 인스턴스: null 로 초기화된다.  

#### 초기화는 왜 필요하지? 
- 자바에서 변수를 생성할 때를 생각해보자

```java
 
 int num1; //선언만 했다. 
 System.out.println(num1);  //초기화가 필요하다고 오류가 뜬다. 
 
 int num2=15; //선언과 초기화를 동시에 한 경우
 System.out.println(num2); //15  잘 나온다.
 
```

- 선언은 어떤 값을 저장하는 공간을 확보하겠다는 것을 의미한다.
- 위 코드를 실행하면 변수가 선언은 되었지만 초기화가 필요하다고 오류가 뜬다. 
- 어떤 변수와 타입을 선언했으면 그에 맞는 값을 넣어줘야 하는데 엉뚱한 타입의 값을 넣는 것을 방지하기 위해 처음에 타입에 맞는 값을 넣어주는 과정(초기화)이 필요한 것이다.

### 생성자가 왜 필요하지?
- 어떤 클래스를 기반으로 만든 인스턴스의 멤버 변수가 초기화가 되지 않고 선언만 되었다.
- 인스턴스를 사용하면서 멤버 변수에 어떤 값이 할당되어 초기화될지 예측할 수 없다. 
- 이를 막기 위해 생성자를 통해 멤버 변수를 초기화하여 변수 타입에 맞는 값을 미리 넣어주는 것이다. 

<br>
## 매개 변수를 포함한 일반 생성자 메소드

```java
public class Person{
	int age;
    String name; //멤버 변수 선언
	
    public Person(int age, String name){ //매개변수 age, name 포함
    	this.age =  age;
        this.name = name; 
        //매개변수 age, name이 this.age, this.name으로 멤버 변수를 가리키고 있다.
    }
}
```

```java
public class PersonTest{
	public static void main(String[] args) {

		//Person personLee = new Person();
        //personLee.age=22;
        //personLee.name="이길동";
		//기본 생성자 메소드로 초기화된 것
        
		Person personLee = new Person(22, "이길동");
        //매개변수를 이용한 생성자

}
```

- this를 이용해 멤버변수를 가리킬 수 있다. 
- 코드를 간결하게 만들 수 있다. 
- 멤버변수의 값을 원하는 값으로 초기화할 수 있다. 
- private으로 클래스의 멤버 변수를 설정하면 초기값 설정을 강제할 수 있다.

<br>
### 생성자 overloading(오버로딩)
- 여러 가지 생성자를 정의하는 것을 의미한다.
- 생성자를 따로 구현하게 되면 기본 생성자 메소드는 제공되지 않는다. 

```java

public class Person{
	int age;
    String name; 
	
    public Person(){} //기본 생성자
    
    public Person(String name){
   		this.name = name;  //이름 매개변수만 있는 생성자
    }
    
    public Person(int age, String name){ 
    	this.age =  age;
        this.name = name;  //나이와 이름 매개변수가 있는 생성자
        
   }
    
   public void printInfo() {
	   System.out.println( "이름:" + name + "나이: " +  age);
   }
}

```

```java

public class PersonTest{
	public static void main(String[] args) {

		Person personLee1 = new Person();
        //기본 생성자로 생성
		Person personLee2 = new Person("이길동");
        //이름 매개변수만 있는 생성자로 생성
		Person personLee3 = new Person(22, "이길동");
        //나이, 이름 매개변수가 있는 생성자로 생성
        
        personLee1.printInfo(); //이름: null 나이: 0
		personLee2.printInfo(); //이름: 이길동 나이: 0
		personLee3.printInfo(); //이름: 이길동 나이: 22
	}
}

```

- 여러 생성자 중 호출하는 사람의 필요에 의해 사용될 수 있다. 
- 매개변수의 타입과 갯수가 같다면 오버로딩이라 할 수 없다. 


<br>
Reference:
- 백엔드 개발 필수_패스트캠퍼스를 학습하고 정리한 내용입니다.   
- [참조형 변수_거신cms](https://colossus-java-practice.tistory.com/7)
- [생성자_TCP](http://www.tcpschool.com/java/java_methodConstructor_constructor)