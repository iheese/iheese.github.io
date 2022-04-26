---
layout: post
title: "[JAVA] JAVA.IO(INPUT, OUTPUT)"
subtitle: " InputStream, OutputStream, Reader, Writer, BufferedReader, BufferedWriter "
date: 2022-04-26 12:00:00 +0900
categories: "JAVA"
background: "/img/posts/javaetc/java.png"
---

![io1](/img/posts/javaetc/io1.png)


![io2](/img/posts/javaetc/io2.png)

- 입력 스트림 :  소스(키보드, 파일, 네트워크 등)를 Java Application으로 데이터를 읽어 들이는 연결
- 출력 스트림 : Java Application에서 목적지(모니터, 파일, 네트워크 등)로 출력하는 연결


### InputStream(입력), OutputStream(출력)
- 두 클래스는 추상 클래스로 상속된 하위 클래스를 이용한다.
- 1 Byte(8Bit) 단위로 입출력을 처리한다.
- 한글 같은 문자를 입출력하면 문자가 깨진다.
- 이미지, 영상 입출력에 이용한다.
- 표준 입출력을 담당하는 클래스 :  java.lang.System
> - InputStream : 표준 입력
> - PrintStream : 표준 출력
> - 표준입출력 데이터형을 가지는 System 클래스의 3가지 변수 
> > - `static InputStream in` : `System.in`, 키보드에서 데이터를 읽어올 때 사용한다.
> > - `static PrintStream out` : `System.out`, 모니터로 데이터를 출력할 때 사용한다, println, print 메소드 제공
> > -  `static PrintStream err` : `System.err`, 모니터로 에러를 출력할 때 사용한다. 거의 사용되지 않는다. 

<br>
### Reader(입력), Writer(출력)
- 두 클래스는 추상 클래스로 상속된 하위 클래스를 이용한다. 
- 2 Byte(16Bit ,Char) 단위로 입출력을 처리한다.
- 대부분 문자 입출력에 사용한다.(한글도 깨지지 않는다.)

<br>
### 파일 입출력
- 파일 입력 : FileInputStream, FileReader를 사용한다. 

- FileInputStream

```java
import java.io.FileInputStream;

public class InputTest {
	public static void main(String[] args) throws IOException {
		FileInputStream fileInput = new FileInputStream("./src/data");
		//같은 프로젝트의 src 파일에 data 라는 파일을 만들었다.
		int read = 0;
		
		while ((read = fileInput.read()) != -1) {
			System.out.print((char)read);
		}  //1 바이트씩 읽어 출력한다.
		fileInput.close(); //입력 스트림을 닫아준다. 
	}
}
		//Hi, my name is Lee! //  영어는 잘 읽어냈지만
		//ìë, ë´ ì´ë¦ì ë¦¬ì¼! // 한글은 읽어내지 못했다. 
```

- FileReader

```java
import java.io.FileReader;
import java.io.IOException;

public class InputTest {
	public static void main(String[] args) throws IOException {
		FileReader fileInput = new FileReader("./src/data");
		
		int read = 0;
		
		while ((read = fileInput.read()) != -1) {
			System.out.print((char)read);
		} //2바이트씩 읽어 출력한다.
		fileInput.close();  //입력 스트림을 닫아준다.
	}
}
		//Hi, my name is Lee!
		//안녕, 내 이름은 리야!  //한글과 영어 모두 잘 읽어냈다. 

```

<br>
- 파일 출력 : FileOutputStream, FileWriter를 사용한다.
- FileOutputStream

```java
import java.io.FileOutputStream;
import java.io.IOException;

public class OutputTest {

	public static void main(String[] args) throws IOException {
		FileOutputStream fileOutput = new FileOutputStream("./src/output1.txt");
		
		byte[] data = {65, 66 , 'A', 'B'};
		fileOutput.write(data);
		fileOutput.write('\n');
		
        fileOutput.write('안');
		fileOutput.write('녕');
		
		fileOutput.close();

	//ABAB
	//HU
	// byte 로 입력하는 숫자는 모두 아스키코드에 의해 자동으로 문자로 변환된다.
	// 영어는 잘 출력되지만 한글은 깨져서 출력되지 않는다.
	}	
}
```

- FileWriter

```java
import java.io.FileWriter;
import java.io.IOException;

public class OutputTest {

	public static void main(String[] args) throws IOException {
		FileWriter fileOutput = new FileWriter("./src/output2.txt");
		
		fileOutput.write(("65");
		fileOutput.write(65);
		
		fileOutput.write('\n');
		fileOutput.write('안');
		fileOutput.write('녕');
		
		fileOutput.write('\n');
		fileOutput.write("반가워요");
		
		fileOutput.close();
	}	
}
		//65A
		//안녕
		//반가워요
        //한글, String을 출력할 수 있다.
        //숫자는 아스키 코드로 반환하여 영어로 출력한다.
		
```

<br>

### BufferedReader, BufferedWriter

- BufferedReader

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class BufferedReaderTest {
	public static void main(String[] args) throws IOException {
			BufferedReader br = new BufferedReader(new FileReader("./src/data"));
			//FileReader 객체를 BufferedReader에 넣어 br을 생성했다.
			String read = null;
			
			while ((read = br.readLine()) != null) {
				String[] nameList = read.split(" ");
				for (String name : nameList) {
					System.out.println("성 : " + name);
				}
			}
			System.out.println("출력이 모두 끝났습니다.");
			
			br.close(); 
            
		}
	}
            //성 : Lee
			//성 : kim
			//  (...)
            //성 : 최
			//성 : 임
			//출력이 모두 끝났습니다.
```

- BufferedReader는 메모리의 버퍼를 생성하여 읽은 데이터를 한번에 모아서 출력하는 역할을 한다. (기본 버퍼 생성자 사이즈: char[8192])
- Scanner를 이용해 입력 받는 방법보다 중간에 버퍼링이 된 후 전달되기 때문에 시스템 데이터 처리 효율성을 높인다.
- BufferedReader의 read() 는 char 단위로 하나씩 읽어 들이고, readLine()은 라인 단위로 읽어 들인다.
> - 라인으로 데이터를 입력 받았을 때는 split, StringTokenizer 등을 통해 데이터를 구분해주어야 한다. 

<br>

- BufferedWriter

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.io.InputStreamReader;

public class BufferedWriterTest {

	public static void main(String[] args) throws IOException {
			BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
			// 키보드로 부터 입력값을 받고 버퍼 과정을 통해 입력 받는다.
            
			BufferedWriter bw = new BufferedWriter(new FileWriter("./src/data.txt"));
			// data.txt 파일을 생성하고 입력한다.
            
			System.out.println("몇 명의 이름을 입력하시겠습니까?");
			int n = Integer.parseInt(br.readLine());
			//입력받을 숫자를 받는다.
			
            System.out.println( n +"명의 이름을 적어주세요 (엔터로 구분해주세요)");
			
			for(int i=1; i<=n; i++) {
				String name =br.readLine();
				bw.write(i +"번 이름 : " +name + "\n");
			}
			//엔터로 구분해서 사람 이름을 받고 파일에 출력한다.
            
            /* 콤마로 구분해서 입력받고 파일 생성하기
			String[] nameList = br.readLine().split(",");
			int i =1;
			for( String name : nameList) {	
				bw.write( i+"번 이름 : " +name + "\n");
				i++;
			}
           */ 
           
			System.out.println("파일 생성 완료!");
            
			br.close(); 
			bw.close();
		}
	}


```

- 버퍼 객체를 생성하고 나서 작업이 끝난 후 꼭 close()를 이용해 닫아주어야 한다.
- 위의 예시의 경우 FileReader, FileWriter 를 버퍼 객체에 넣어 생성했기 때문에 버퍼 객체만 닫아주었지만 따로 생성했다면 FileReader, FileWriter을 먼저 닫아주고 버퍼 객체를 닫아주면 된다.
> - BufferReader의 경우 close()하지 않아도 Garbage Collector에 의해 내부 객체가 정리가 된다.
> - BufferWriter의 경우 적당한 시점에 해주지 않으면 원치 않는 것이 출력될 수 있다.
>> - flush() : 메소드는 버퍼에 남아있는 데이터를 강제로 내보내는 기능이다.
>> - close() : flush()를 통해 버퍼의 남아있는 데이터를 내보내고 버퍼를 닫는다.
>>> - 자원 관리를 위해 두 메소드를 적재적소에 이용할 필요가 있다. 

<br>

Reference:
- [java img_크레벅스](https://www.crebugs.com/product/view.php?idx=7382&code=1412)  
- [[Java] 자바 I/O 에 대한 이해 (1)(2) img_GRUNBI](https://ccm3.net/archives/21118)
- [입출력 관련 백준 문제 풀이_Modest Study](https://ddungi.github.io/algorithm/2022/04/23/bj_15552/)
