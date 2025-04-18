---
layout: post
title: '[JAVA] 백준_11728, 2164, 9012'
subtitle: 'ArrayList, Queue, Stack 문제 풀이'
date: 2022-04-14 12:00:00 +0900
categories: [algorithm]
background: '/img/posts/etc/datastructure.jpg'
use_math: true
---


## 11728 배열 합치기 
- ArrayList 이용하여 푸는 문제


```java
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Main {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int N = sc.nextInt();
		int M = sc.nextInt(); //각각 리스트 갯수 2개를 받아준다. 
		
		List<Integer> A = new ArrayList<>(); 
		for(int i = 0; i < N; i++) {
			int n = sc.nextInt();
			A.add(n);  //첫번째 리스트 생성
		}
		List<Integer> B = new ArrayList<>();
		for(int i = 0; i < M; i++) {
			int m = sc.nextInt();
			B.add(m);  //두번째 리스트 생성
		}
		
		List<Integer> result = new ArrayList<Integer>();
						// 합칠  리스트 생성
		int i =0; int j =0;
		while (i <N && j< M) {
			int a = A.get(i);
			int b = B.get(j);   
            //두 리스트 중 하나라도 모두 result에 채워지면 멈춘다.
			
			if(a<=b) {
				result.add(a); 
				i++;
			}else {
				result.add(b);
				j++;
			}
		}//하나씩  비교하면서 작은 순서대로 넣어준다.
		
		for (; i<N;i++) {
			result.add(A.get(i));
		} 
		
		for (; j<M;j++) {
			result.add(B.get(j));
		}
        // 둘 중에 나머지 값은 그냥 붙여준다. 
        
		
		StringBuilder sb =new StringBuilder();
		for (int e :  result) {
			sb.append(e+" ");
		}
		System.out.println(sb.toString());
		// 성능이 좋은 가변길이 StringBuilder 이용
	}

}

```


<br>
## 2164 카드2
- Queue를 이용하여 푸는 문제


```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.Scanner;

public class Main {

	public static void main(String[] args) {

		Scanner sc = new Scanner(System.in);
		int N =sc.nextInt();
		// 카드 갯수를 입력 받는다. 1이 제일 위로 오게 카드를 쌓는다.

		// 큐는 링크드리스트로 구현한다. 
		Queue<Integer> queue =new LinkedList<>();
		for(int i = 0;i<N;i++) {
			queue.add(i+1);
		}
        
		int count =1 ;
        // 큐의 크기가  1이 되면 반복문이 끝난다.
		while(queue.size() != 1) {
			int q = queue.poll();
			if (count % 2==0) {
				queue.offer(q);
			} // 첫번째꺼 버리고 두번째꺼 맨 아래로 
			count++;
		}
		System.out.println(queue.peek());
		// 마지막 남은 카드를 출력한다. 
		
	}

}
```


## 9012 괄호
- Stack을 이용하여 푸는 문제


```java
import java.util.Scanner;
import java.util.Stack;

public class Main {

	public static void bracket(String s) {
		Stack<Character> stack = new Stack<>();
        //스택 객체 생성

		int i = 0;
		while (i < s.length()) {
			char c = s.charAt(i);
			// 입력 받은 문자열을 하나씩 확인한다. 
			if (c == '(') {
				stack.push(c);
                
			}else {
				if (stack.size() < 1) {
					System.out.println("NO");
					return;
                    아무것도 없을 때 ')'이걸 넣으면 NO 
				}
				stack.pop();
			} //'(' 이면 쌓고 다른게 나오면 빼는 함수이다.
			i++;
		}
			if (stack.size()>0){
				System.out.println("NO");
			} else {
				System.out.println("YES");
			}
		//그러다가 양수가 되면 NO, 아니면 YES
	}

	public static void main(String[] args) {
		
		Scanner sc = new Scanner(System.in);
		int T = sc.nextInt();
			// 갯수를 입력 받고 각각 문자열을 함수로 확인한다. 
		for (int i = 0; i < T; i++) {
			bracket(sc.next());
		}

	}
}
```

<BR>
Reference:
- [11728 배열 합치기_백준](https://www.acmicpc.net/problem/11728)
- [2164 카드2_백준](https://www.acmicpc.net/problem/2164)
- [9012 괄호_백준](https://www.acmicpc.net/problem/9012)
