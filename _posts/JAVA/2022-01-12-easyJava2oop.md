---
layout: post
title: '쉽게 배우는 자바2'
subtitle: '객체 지향 프로그래밍(OOP: Object Oriented Programming)'
date: 2022-01-12 11:00:00 +0900
categories: 'JAVA'
---

*모든 실습은 eclipse에서 진행되고 있다.

## 객체 지향 프로그래밍(OOP: Object Oriented Programming)

-  관련있는 변수와 메소드를 모은 수납상자 같은 클래스로 프로그램을 정리정돈
- 객체 지향 언어: 언어적 차원에서 이를 지원하는 언어

```java
import java.io.FileWriter;
import java.io.IOException;

public class OthersOOP {

    public static void main(String[] args) throws IOException {
        // class : System, Math, FileWriter
        // instance : f1, f2
            FileWriter f1 = new FileWriter("data.txt"); //인스턴스 생성
        f1.write("Hello");
        f1.write(" Java");
         
        FileWriter f2 = new FileWriter("data2.txt"); //인스턴스 생성
        f2.write("Hello");
        f2.write(" Java2");
        f2.close();
         
        f1.write("!!!");
        f1.close();
    }
 
}
       
```
- 인스턴스를 생성할 수 없는 클래스도 존재한다.(Ex: Math)
- Math클래스 경우 내부적으로 어떤 상태를 계속 유지할 필요는 없다.(일회용처럼 사용하면 된다.)
- FileWriter 클래스는 계속 작업을 추가할 수 있어 이에 유리한 인스턴스를 사용하게 제공된다. 

<br>

### class 만드는 방법
- 같은 java 파일 내 class를 만들 수도 있다. 
> - 만들고 싶은 범위 드래그-오른쪽 클릭- Refactor - Extract Method (Alt+Shift+M)
- class만 다른 파일로 빼내기
> - 원하는 class 드래그- Refactor - Move Type to New File

<br>

## class와 instance

```java
class Print {
	public String delimiter = "";
	public void A() {  //클래스 내 메소드 두 개 생성
		System.out.println(delimiter);
		System.out.println("A");
		System.out.println("A");
	}
	public void B() {
		System.out.println(delimiter);
		System.out.println("B");
		System.out.println("B");
	}
}
public class MyOOP {
	
	public static void main(String[] args) {
		Print p1 = new Print();
		p1.delimiter = "----";
		Print p2 = new Print();
		p2.delimiter = "****";  
        //인스턴스가 있으면 쉽게 변수도 바꿀 수 있다.
		
		p1.A();
		p2.A();  //인스턴스가 있으면 쉽게 메솓즈를 호출할 수 있다. 
		p1.B();
		p2.B();	
	}	
}
```

- 클래스의 복제본인 인스턴스는 같은 클래스를 다른 데이터 값으로 사용하고 싶을 때 사용 가능 하다. 

<br>

### static
```java
class Foo{
    public static String classVar = "I class var";
    public String instanceVar = "I instance var";
    public static void classMethod() {
        System.out.println(classVar); // Ok
//      System.out.println(instanceVar); // Error
    }
    public void instanceMethod() {
        System.out.println(classVar); // Ok
        System.out.println(instanceVar); // Ok
    }
}
public class StaticApp {
 
    public static void main(String[] args) {
        System.out.println(Foo.classVar); // OK
//      System.out.println(Foo.instanceVar); // Error
        Foo.classMethod();
//      Foo.instanceMethod();
         
        Foo f1 = new Foo();
        Foo f2 = new Foo();
//      
        System.out.println(f1.classVar); // I class var
        System.out.println(f1.instanceVar); // I instance var
//      
        f1.classVar = "changed by f1";
        System.out.println(Foo.classVar); // changed by f1
        System.out.println(f2.classVar);  // changed by f1
//      
        f1.instanceVar = "changed by f1";
        System.out.println(f1.instanceVar); // changed by f1
        System.out.println(f2.instanceVar); // I instance var
    }
 
}
````

- static 변수와 메소드는 클래스에서 생성된 모든 인스턴스가 공유하는 자원
- static이 아닌 변수와 메소드는 인스턴스를 생성해야 접근할 수 있다.
- static이 아닌 변수와 메소드는 각 고유의 값을 가지므로 인스턴스에서 변경한다 해도 다른 인스턴스와 클래스에 영향을 주지 않는다.
- 어떤 클래스에서 다른 인스턴스를 만들 때 고정시키고 싶은 값은 static을 붙인다. 
- 어떤 클래스에서 다른 인스턴스를 만들 때 다양한 값이 들어올 것 같은 것, 변화를 주며 사용할 것 같은 것은 static을 붙이면 안된다. 

<br>

### 생성자, this
```java
class Print {
	public String delimiter = "";
	public Print(String delimiter) {
		this.delimiter = delimiter; //-------1번
	}
    // this는 각 인스턴스를 가리키는 예약어
    public Print(String _delimiter) {
		delimiter = _delimiter; //-------2번
	}
    
    // 1번 2번 둘 다 가능하다. 
	public void A() {
		System.out.println(delimiter);
		System.out.println("A");
		System.out.println("A");
	}
	public void B() {
		System.out.println(delimiter);
		System.out.println("B");
		System.out.println("B");
	}
}

public class MyOOP {
    public static void main(String[] args) {
        Print p1 = new Print("----");
        p1.A();
        
        Print p2 = new Print("****");
        p2.B();
        }
}
```

- 클래스는 따로 만들어주지 않아도 기본 생성자를 포함한다.(Print() 이와 같은)
- 클래스와 같은 이름으로 메소드를 만들면 생성자를 만들 수 있다.
- 생성자는 접근 권한을 설정할 수 있고, 리턴 타입은 명시하지 않으며, 초기화할 필드에 따라 파라미터를 설정한다. 

<br>

Reference:

- JAVA 객체지향 프로그래밍, 쉽게하는 자바2_생활코딩, 부스트코스 과정을 통해 공부한 내용입니다.