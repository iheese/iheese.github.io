---
layout: post
title: '[JAVA] 백준_2884, 2525, 2480'
subtitle: '조건문(conditional)'
date: 2022-04-22 12:00:00 +0900
categories: 'Algorithm'
---

## 2884

```java
import java.util.Scanner;

class Main{
  public static void main(String[] args){
    Scanner sc = new Scanner(System.in);  
      int H= sc.nextInt();
      int M= sc.nextInt();
		
	  	if(M < 45) {
	  		H--;		// 시(hour) 1 감소
	  		M= 60 - (45 - M); 	// 분(min) 감소
	  		if(H < 0) {
				H = 23;
	  		}
	  		System.out.println(H + " " + M);
	  	}
	  	else {
			System.out.println(H + " " + (M - 45));
	    	}
        }
   }
```

<br>
## 2525

```java
import java.util.Scanner;

public class BJ2525 {
	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int h = sc.nextInt();
		int m = sc.nextInt();
		int later =sc.nextInt();
		
		int minutes= m+later;
		h = h+(minutes/60);
		minutes = minutes%60;
			if(h>=24) {
				System.out.println((h-24)+" "+minutes);
			}else {
				System.out.println(h+" "+minutes);
			}
		
		
	}
}

```

<br>

## 2480

```java

import java.util.Scanner;

public class BJ2480 {

	public static void main(String[] args) {
		Scanner sc =new Scanner(System.in);
		int a =sc.nextInt();
		int b =sc.nextInt();
		int c =sc.nextInt();
		
		if(a==b && b==c) {
			System.out.println(10000+a*1000);	
		}else if(a!=b && b!=c && a!=c ) {
			int max = a;
			if(b>max) max=b;
			if(c>max) max=c;
			System.out.println(max*100);
		}else {
			int same = ((a==b)? a : c);
			System.out.println(same*100+1000);
		}
	}

}
```

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int a = sc.nextInt();
        int b = sc.nextInt();
        int c = sc.nextInt();
        if(a == b && b == c)
            System.out.print(10000 + a*1000);
        else if(a == b || a == c)
            System.out.print(1000 + a*100);
        else if(b == c)
            System.out.print(1000 + b*100);
        else
            System.out.print((Math.max(Math.max(a, b), c)*100));
    }
}
```

Math.max도 이용할 수 있다. 




<br>
Reference:
- [2480_ OTOTOT](https://imototot.tistory.com/342)