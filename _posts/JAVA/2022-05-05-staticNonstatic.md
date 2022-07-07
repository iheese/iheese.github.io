---
layout: post
title: '[JAVA] 정적 메소드와 비정적 메소드의 저장 방식과 동작 방식, JVM 메모리와 운영체제 메모리'
subtitle: 'JVM, Static , Non-static, Virtual Method, Process Memory Structure'
date: 2022-07-07 12:00:00 +0900
categories: 'JAVA'
background : '/img/posts/javaetc/java.png'
---
 
 
- (22/07/07)
> - 정적 메소드와 비정적 메소드에 대한 공부를 하던 중에 올렸던 게시물이다. 더 학습하면서 보완할 점이 있어 업데이트를 한다. 
> - 쏟아지는 정보 속에서 JVM의 메모리와 운영체제가 제공하는 메모리가 헷갈렸었다. 둘은 비슷하지만 다르며 게시물 아래 운영체제의 메모리를 덧붙여보겠다.
- (22/05/05)
> - 알고리즘 문제를 풀 때 한 클래스에서 함수를 만들어 호출하여 사용할 때 static 메소드를 사용하는 이유는 뭘까?
> - main 메소드는 static 메소드인데, main 메소드에서 비정적 메소드를 호출하지 못하는 이유는 무엇일까?


<BR>

```java
public class StaticTest {
		public static void main(String[] args)  {
			Static(); //I'm Static
			nonStatic(); //오류가 뜬다.
        }
        
		 public static void Static(){
	        	System.out.println("I'm Static");
	    }
		
        public void nonStatic(){
        	System.out.println("I'm non-Static");
        }
    }
```

- [`STATIC` 에 관한 기본적 내용](https://ddungi.github.io/java/2022/04/12/static/)

#### non-static method cannot be referenced from a static contex

- <a href="#target">참조글(stackoverflow)</a>의 글을 바탕으로 이해해보자
- Non-static 메소드는 객체 생성 전에는 존재하지 않기 때문에 존재하지 않는 것을 부르는 것은 불가하다. 
- Static 메소드는 클래스에 속해있고, Non-static 메소드는 클래스의 인스턴스에 속해있다. 

<br>
- 더 이해하기 위해 클래스, 스택, 힙을 중심으로 JVM 저장 방식이 어떻게 되는지 알아보자.

<br>

![staticnonstatic](/img/posts/javaetc/staticnonstatic1.png)

![staticnonstatic](/img/posts/javaetc/staticnonstatic2.png)

## JVM(Java Virtual Machine) 메모리
 
- 런타임이 시작하면 JVM은 용도에 맞게 영역에 메모리를 할당한다.
- 클래스 영역(메소드 영역) 
> - 멤버 변수에 대한 정보, 메소드의 데이터, static 변수와 메소드, 상수풀, 런타임에  필요한 메소드, 필드 참조 등의 상수까지 포함된다.
> - 위 값은 프로그램 시작시 생성되고 종료와 함께 소멸된다.
- 스택 영역
> -  메소드 호출시 각각의 지역변수, 매개변수 (참조변수형), 리턴 값 등이 프레임 형태로 저장되었다가 사용된 뒤 사라진다.
> - 원시 데이터(int, double, boolean 등) 도 저장된다.
- 힙 영역 
> - 객체(Object 상속받은 것들)들과 배열을 저장하는 공간
> - new 생성자에 의해 동적으로 생성된다. 
> - 가비지 컬렉터에 의해 사용하지 않는 데이터는 자동으로 제거된다. 
> > - New, Old 공간이 있어 New에서 객체가 생성되어 저장되어 있다가 일정 시간이 지나면 Old 지역으로 넘어간다.
- Native 메소드 영역
> - JVM는 Java가 아닌 다른 언어로 작성된 코드를 지원하고 이와 같은 데이터를 저장하는 영역이다.
- PC 레지스터
> - Thread가 생성될 때마다 생성되는 공간이며 어떤 부분(주소)을 어떤 명령으로 실행할 지에 대한 기록들이 저장된다.
  
<br>

### Static 메소드

- 우리가 자주 사용하는 main 메소드도 static이므로 런타임이 시작될 때 할당받으면서 클래스 영역에 속하게 된다.
- 또한 static이라 선언한 메소드는 런타임이 시작되면서 클래스 영역에 할당되기 때문에 바로 접근하여 실행이 가능하다.
> - 같은 클래스 내 static 메소드는 `클래스.메소드` 형식으로 부르지 않아도 되고 그냥 메소드명으로 호출한다. 
> - `클래스.메소드` 형식을 이용하면 다른 클래스에서도 접근이 가능하다.(접근제어자가 가능한 수준일 때)

<br>
## 내가 작성한 Non-static 메소드는 어디에 있는가?

- Non-static 메소드(명령어 집합)는 클래스 영역에 위치하게 된다.
- 하지만 Non-static 메소드는 클래스의 인스턴스화가 이뤄지지 않으면 클래스의 변수도 접근할 수 없기 때문에 그냥 `명령어 집합`으로 남아있게 된다.

- 기본적으로 java의 메소드는 모두 가상 메소드 방식을 가진다.
> - 메소드의 이름은 주소값을 나타낸다. 
> - java의 오버라이딩은 같은 이름이지만 다른 메소드를 칭하기도 한다.


- 인스턴스를 생성하게 되면 힙 영역에 객체가 생성되고 객체는 스택 영역에 지역 변수를 만들어 이용하거나, 다른 객체의 필드를 참조하며 실행된다. 
> - 같은 클래스로 만들어진 이름이 다른 인스턴스도 같은 메소드를 사용한다.  
> - 오버라이딩했을 경우 클래스 영역에 새로운 메소드가 따로 생성된다.

<br>

![virtual function](/img/posts/javaetc/virtualfunc.png)


```java
class Person {
		public void eat() {
			System.out.println("냠냠");
		}
	}

public class PersonTest {
		public static void main(String[] args)  {
			
			Person A = new Person();
			Person B = new Person();
			
			A.eat(); //냠냠
			B.eat(); //냠냠
    }
}
```

- 위의 eat()는 같은 메소드를 호출하여 사용한다. 

<br>
## JVM이 이런 방식을 선택한 이유는 뭘까?

- 객체 지향 프로그램의 가장 중요한 목적은 **유지보수와 재사용성**에 있다.
> - 매번 인스턴스를 생성하여 메소드를 사용하는 방식은 자주 사용하거나 쉽게 접근해야하는 메소드에는 적합하지 않다.
> - 모든 메소드가 쉽게 접근이 가능하다면 메모리 낭비와 정보 은닉 등의 문제가 생길 것이다.

<br>
### 마무리
- 정적 메소드는 런타임이 진행될 때 생성된 메소드이므로 바로 접근하여 사용이 가능하다.
> - 정적 메소드 생성을 남발하는 것은 프로그램 자체에 메모리 낭비, 정보 은닉 등에 문제가 생길 수 있다. 
> - 알고리즘 문제 풀이 경우 메소드만 필요한 경우 생성하여 바로 사용하므로 정적 메소드를 이용하는 것이 효율적이다.
- 비정적 메소드는 명령어 집합으로만 존재하므로 인스턴스 생성을 통해 접근이 가능하다.

<br>

### 운영체제의 메모리(프로세스 구조)

![memory](/img/posts/cs/memory.png)

- 운영체제가 제공하는 메모리는 코드영역, 데이터 영역, 힙 영역, 스택 영역으로 나뉜다.
- 코드 영역(텍스트 영역)
> - 실행할 프로그램의 코드가 저장되는 곳이며 CPU가 코드 영역의 명령어를 하나씩 처리한다.

- 데이터 영역
> - 전역변수와 정적(static) 변수가 저장되는 공간이며 프로그램의 시작과 함께 할당되고 프로그램이 종료되는 순간 소멸된다.
- 힙 영역
> - 코드를 통해 동적으로 만들어지는 데이터가 저장된다. 
> - c언어의 경우 사용자가 직접 관리하기도 하며, Java에서는 가비지 컬렉터
- 스택 영역
> - 임시적인 데이터가 저장되는 곳으로 지역 변수와 매개 변수가 저장된다.
> - 함수의 호출과 함께 할당되고 호출이 완료되면 소멸된다.
가 관리해준다.
> - 파이썬 언어의 재귀함수가 일정 횟수를 넘어가면 에러가 나는 이유는 스택 저장 공간이 정해져 있기 때문이다.

<br>

Reference:
- [메모리의 구조_TCP school, img](http://www.tcpschool.com/c/c_memory_structure)
 
- [What is the reason behind "non-static method cannot be referenced from a static context"?_stackoverflow](https://stackoverflow.com/questions/290884/what-is-the-reason-behind-non-static-method-cannot-be-referenced-from-a-static) <a id='target'></a>
- [JVM 메모리구조_어떤공간](https://huelet.tistory.com/entry/JVM-%EB%A9%94%EB%AA%A8%EB%A6%AC%EA%B5%AC%EC%A1%B0)
- [가상 함수(virtual function)에 대하여_place of 42seoul story](https://42place.innovationacademy.kr/archives/8728)
