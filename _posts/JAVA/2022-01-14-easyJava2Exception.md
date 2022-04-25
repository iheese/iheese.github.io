---
layout: post
title: '[JAVA] 쉽게 배우는 자바2'
subtitle: '예외 (Exception)'
date: 2022-01-14 12:00:00 +0900
categories: 'JAVA'
---
*모든 실습은 eclipse에서 진행되고 있습니다. 

## 예외(Exception)
- 예상한 범위를 벗어나는 경우
- 잘 대처하면 피할 수 있다.
- Ex) 생각한 파일이 존재하지 않는 경우, 0으로 나눗셈을 하는 경우, 배열에서 인덱스를 벗어나 호출한 경우 등
> - 오류(Error): 피할 수 없는 경우 
> - Ex) 문법 오류로 인해 컴파일 오류, 메모리 부족, 정전 등

### 예외 처리 방법
```java
public class ExceptionApp {
	public static void main(String[] args) {
		System.out.println(1);
		int[] scores = {10, 20, 30};

		try {
			System.out.println(2);
			System.out.println(scores[3]);
            //ArrayIndexOutOfBoundsException
			// 여기까지 실행 ->
            // catch(ArrayIndexOutOfBoundsException e)문으로 이동
			System.out.println(3);
			System.out.println(2/0); //ArithmeticException
			System.out.println(4);
		} catch(ArithmeticException e) {
			System.out.println("잘못된 계산이네요.");
		} catch(ArrayIndexOutOfBoundsException e) {
			System.out.println("없는 값을 찾고 계시네요~");
		}		
		System.out.println(5); 
	}
}
//1
//2
//없는 값을 찾고 계시네요~
//5
```
- try,catch 문으로 예외를 처리할 수 있다.
- try 중 예외가 발생하면 바로 그에 맞는 catch의 명령문으로 가고 catch의 중괄호를 벗어나 다음 명령어를 처리한다. 
- 즉 try 명령문 내 예외 아래 있는 명령문은 실행되지 못한다.
<br> 
- 예외 처리도 클래스로 구현되어 있고 서로 상속되어 있다.
- Ex) Exception(부모 클래스) - RuntimeException -ArithmeticException(자식 클래스)
- ArithmeticException 같은 경우 Exception으로 포괄적으로 처리해줄 수도 있다.
- 예외가 발생했을 때 순서대로 catch문을 검사하며 내려가므로 순서도 중요하다. 
- 포괄적인 예외처리는 구체적으로 어떤 예외 상황인지 알기 어려운 단점이  있다. 
<br>

#### e 변수
```java
public class ExceptionApp {

	public static void main(String[] args) {
		System.out.println(1);
		int[] scores = {10,20,30};
		try {
			System.out.println(2);
			System.out.println(3);
			System.out.println(2/0);
			System.out.println(4);
		} catch(ArithmeticException e) {
			System.out.println("잘못된 계산이네용!!!!!"+e.getMessage());
			e.printStackTrace();
            }
            System.out.println(5);
        }
    }
//1
//2
//3
//잘못된 계산이네용!!!!!/ by zero      0으로 나눈 것이 잘못되었다고 한다. 
//java.lang.ArithmeticException: / by zero
//	at ExceptionApp.main(ExceptionApp.java:11)
//5
    
```
- catch문의 변수 e는 인스턴스이다.
- e.getMessage()를 이용하면 간단한 예외상황에 대한 정보를 알 수 있다. 
- e.printStackTrace()를 이용하면 자세한 예외 상황에 대한 정보를 알 수 있다. 
- 하지만 이러한 정보는 코드의 내용과 구조를 공개할 수 있기 때문에 서버 측에서 로그 파일 등을 통해 관리자만 볼 수 있게 처리한다. 
<br>
### Checked, Unchecked Exception

![checked, unchecked Exception](/img/posts/easyjava/Exception.png)

- Checked Exception: 반드시 예외 처리를 해줘야 하는 예외
> - RuntimeException으로 부터 상속된 예외 말고 모두
- Unchecked Exception: 컴파일 실행은 가능한 예외, 콘솔에서 예외 발생 시 표시를 해준다. 
> - Throwable-Exception -RuntimeException 부터 상속된 모든 예외
> - 이는 컴파일 시는 발생하지 않고 실행 도중에 발생한 예외이다. 
<br>

### Finally, 자원(Resource)
- 자원(Resource): 파일, 네트워크, 데이터베이스 같은 프로그램 외부의 것들
- 외부의 자원은 불안정하기 때문에(우리의 자바 프로그램만을 위한 것이 아니므로) 점유하고, 연결상태를 유지한 후 자원을 놓아주는 작업이 필요하다. 

```java
import java.io.FileWriter;
import java.io.IOException;

public class CheckedExceptionApp {

	public static void main(String[] args) {
		FileWriter f =null;
		try {
			f =new FileWriter("data.txt");
			f.write("Hello");
			//close() 하기 전에 예외 발생하면  close가 실행되지 않음
		} catch(IOException e) {
			e.printStackTrace(); 
		} finally { //예외가 발생해도 무조건 finally는 실행된다.
			//만약에 f가 null이 아니라면
			if (f != null) {
				try {
				f.close();
				} catch(IOException e) {
					e.printStackTrace();
				}
				}
		}
	}
}
```
### try with resource 
- 위는 조금 복잡한 코드가 될 수 있다.
- AutoCloseable 인터페이스를 상속하면 try with resource 문을 사용할 수 있다.
- Java SE 7 이상에서 작동한다.

```java
import java.io.FileWriter;
import java.io.IOException;

public class TryWithResource {

	public static void main(String[] args) {
		try (FileWriter f = new FileWriter("data.txt")) {
			f.write("Hello");
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
} //close() 없이도 자동으로 종료된다. 
```
<br>

### throw 
```java
public class MyException {

	public static void main(String[] args) {
		throw new RuntimeException("런타임 문제가 있습니다.");
	}
	public static void main(String[] args) {
		throw new Exception("뭔가 문제가 있습니다.");
	}
}   
```
- 예외를 인스턴스 생성처럼 객체로 생성할 수도 있다. 

### throws

```java
import java.io.IOException;

public class ThrowException {

	    public static void error() throws IOException {
		throw new IOException("입출력 오류가 error 메소드에서 나왔습니다.");
	}
    
 	   public static void main(String[] args) {
			ThrowException sample =new ThrowException();
            try{
            	ThrowException.error();
            } catch (IOException e) {
            	System.out.println("입출력 오류가 main으로 넘어왔습니다.");
            }
	}

}
```
- throws는 예외 상황이 발생하면 그 메소드를 사용하는 메소드로 책임을 전가한다. 
- main 메소드에서 try, catch문으로 예외를 처리하면 중간에 예외 발생으로 인해 실행되지 않는 명령문이 존재할 수 있다.
- 만약 throws를 사용하지 않고 error 메소드에서 예외를 처리하면 그 메소드를 불러 사용하는 main 메소드는 예외가 발생하더라도 모든 명령문이 실행된다.
<br>
#### + 트랜잭션(Transaction)
- 하나의 작업 단위
- Ex) 상품발송이라는 트랜잭션이 있다면 그 속에는 포장, 영수증발행, 발송의 작업이  있다. 세 가지 작업 중 하나라도 실패하면 모두 취소하고 상품 발송 전 상태로 되돌리고 싶어할 것이다. (Rollback)이라  한다. 


```java
//나쁜예
상품발송() {
    포장();
    영수증발행();
    발송();
}

포장(){
    try {
       ...
    }catch(예외) {
       포장취소();
    }
}

영수증발행() {
    try {
       ...
    }catch(예외) {
       영수증발행취소();
    }
}

발송() {
    try {
       ...
    }catch(예외) {
       발송취소();
    }
}
````
- 각각의 메소드에 예외처리가 되어 있으면 포장은 되었는데 영수증 발행은 안되고 발송은 되는 엉터리 트랜잭션이 된다.

```java
//좋은 예
상품발송() {
    try {
        포장();
        영수증발행();
        발송();
    }catch(예외) {
        모두취소();  // 하나라도 실패하면 모두 취소한다.
    }
}

포장() throws 예외 {
   ...
}

영수증발행() throws 예외 {
   ...
}

발송() throws 예외 {
   ...
}
```
- 각 메소드에서 예외를 main()(상품 발송) 메소드에서 처리하게 되면 하나라도 실패하면 바로 상품 발송이 취소된다. 
- 예외 처리가 진행되는 메소드를 잘 이해하고 트랜잭션의 흐름을 잘 파악하는 것이 중요한 것 같다. 
- 독립적으로 예외 처리해야하는 메소드와 묶어서 해결되는 것이 효율적인 메소드들을 잘 구분해야겠다. 

<br>

Reference:
- 쉽게 배우는 자바2_생활코딩, 부스트코스 과정을 학습한 내용입니다.
- [checked,unchecked Exception_ wusi_univ.log](https://velog.io/@wusi_univ/Java-%EC%98%88%EC%99%B8-checked-vs-unchecked-exception)
- [예외처리(Exception)_점프 투 자바](https://wikidocs.net/229)
