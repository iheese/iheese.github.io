---
layout: post
title: '[JAVA] 자료 구조1 (Data Structure)'
subtitle: '큐(Queue), 스택(Stack), 링크드 리스트(Linked list), 시간 복잡도(Time complexity)'
date: 2022-04-08 12:00:00 +0900
categories: 'Algorithm'
background: '/img/posts/etc/datastructure.jpg'
use_math: true
---

*tip: [Linked List, Queue, Stack ](https://visualgo.net/en/list) 직접 실행하면서 이해하는 사이트

<br>
## Queue(큐)
- 가장 먼저 들어간 데이터가 먼저 나오는 방식, 일렬로 줄서는 것과 같다.
- FIFO(First-In, First-Out) 또는 LILO(Last-In, Last-Out) 방식
- Enqueue : 큐에 데이터를 넣는다.
- Dequeue : 큐에서 데이터를 꺼낸다. 


```java
import java.util.LinkedList;  //Queue 사용을 위해 Linked List를 사용한다. 
import java.util.Queue; //Queue import

class Main {
  public static void main(String[] args) {
    Queue<Integer> queueInt = new LinkedList<Integer>(); // Integer 형 queue 선언
    Queue<String> queueStr = new LinkedList<String>(); //Stigng queue 선언

    queueInt.add(1); //큐에 첫 번째 값 넣기 
    queueInt.offer(2); //큐에 두 번째 값 넣기
    queueInt.offer(5); //큐에 세 번째 값 넣기  //Enqueue

    System.out.println(queueInt); // [1,2,5]

    queueInt.poll(); //첫 번째 값 삭제 [2,5]
    queueInt.remove(); //첫 번째 값 삭제 [5] //Dequeue
    
    queueInt.peek(); //큐의 첫번째 값 참조
  
  }
}
```

<br>
#### 큐는 언제 사용될까?
- 큐는 주로 데이터가 시간 순서대로 처리되어야 할 때 사용된다.
> - 프로세스 관리
> - 너비 우선 탐색 구현
> - 주문 처리 과정
> - 메세지 처리기

<br>

## Stack(스택)
- 가장 먼저 들어간 데이터가 제일 늦게 나오는 방식, 책 쌓기와 같다. 
- LIFO(Last In, Fisrt Out) 또는 FILO(First In, Last Out) 방식
- 장점: 구조가 단순, 데이터 저장과 조회가 쉽다.
- 단점: 데이터의 최대 갯수를 미리 정해야 해서 저장 공간의 낭비가 생길 수 있다. 

```java
import java.util.Stack; //stack import

class Main {
  public static void main(String[] args) {
    
    Stack<Integer> stackInt = new Stack<Integer>(); //Integer 형 스택 생성

    stackInt.push(1); //첫 번째 값 입력
    stackInt.push(2); //두 번째 값 입력
    stackInt.push(3); //세 번째 값 입력

    System.out.println(stackInt); //[1,2,3]

    stackInt.pop(); //마지막에 들어왔던 값부터 빠진다.
    System.out.println(stackInt); //[1,2]

    stackInt.pop();
    System.out.println(stackInt); //[1]

    stackInt.pop();
    System.out.println(stackInt); //[]
  }
}

```

#### 스택은 언제 사용될까?
- 스택은 주로 데이터가 시간 역순으로 처리되어야 할 때 사용된다.
> - 웹 브라우저 방문 기록 (뒤로 가기)
> - ctrl+z (undo) 실행 취소

<br>

## Linked List(링크드 리스트, 연결 리스트)
- 각 노드가 데이터와 포인터를 가지고 한 줄로 연결되어 데이터를 저장하는 방식
- Node(노드) 라는 덩어리 안에 데이터와 포인터가 있다.
- 각 노드의 포인터가 다음 노드를 연결한다.(포인터에는 다음 노드의 주소값이 저장된다.)

```java
import java.util.LinkedList; //LinkedList import

class Main {
  public static void main(String[] args) {
    LinkedList<Integer> linkedlistInt = new LinkedList<Integer>(); //Integer 연결리스트 생성
    LinkedList<String> linkedlistStr = new LinkedList<String>();
    
    linkedlistInt.add(3); //순서대로 데이터 3 추가
    linkedlistInt.addFirst(1); //가장 앞에 1 추가
    System.out.println(linkedlistInt); //[1,3]
    
    linkedlistInt.addLast(5); //가장 뒤에 5 추가
    System.out.println(linkedlistInt); //[1,3,5]

    linkedlistInt.add(1,10); //1번 인덱스에 10 추가
    System.out.println(linkedlistInt); //[1,10,3,5]


    linkedlistStr.add("일번");
    linkedlistStr.add("삼번");
    linkedlistStr.add("사번");
    linkedlistStr.add(1, "이번");
    System.out.println(linkedlistStr); // [일번, 이번, 삼번, 사번]
    
    linkedlistInt.removeFirst(); //첫번째 데이터 삭제
    linkedlistInt.removeLast(); // 마지막 데이터 삭제
    linkedlistInt.remove(); // 안에 숫자 넣으면 해당 인덱스 삭제, 빈 값이면 0번째 삭제
    linkedlistInt.clear(); //모든 값 제거
  }
}
```


#### 링크드 리스트는 언제 사용될까
- 배열은 미리 데이터 공간을 할당해야하는 반면 연결 리스트는 그럴 필요가 없다. 
- 연결 리스트 특성상 중간 데이터를 삭제하고 삽입하는 것이 배열에 비해 용이하다. 그러나 특정 요소에 접근하기 위해서는 순차 탐색이 필요하다.(데이터추가와 삭제: O(n), 탐색 시간 복잡도: O(n))
- 데이터를 탐색하는 일에 비해 데이터를 추가하거나 삭제할 일이 많은 경우 사용하면 좋다. 

<br>

### 공간 복잡도 (Space complexity)
- 알고리즘이 수행하는데 쓰이는 메모리 사이즈

## 시간 복잡도(Time complexity) 
- 알고리즘 수행의 실행 속도
- 반복문이 대부분 시간 복잡도를 결정한다. 
- 시간 복잡도 표기법
> - O(n)(빅 오) : 알고리즘 최악의 실행 시간
> > - 최악의 시간을 고려하더라도 이 정도의 성능을 보장한다는 의미한다.
> > - 가장 일반적으로 많이 사용되는 표기법이다.
> > - Ex) O(1), O($\log$ n), O(n), O($\log$ n), O($n^2$), O($2^n$), O(n!) (log의 베이스는 2)
> - $\Omega$(n) : 알고리즘 최상의 실행 시간
> - $\theta$(n) : 알고리즘 평균 실행 시간 


#### 시간 복잡도 계산하기

```java
	for(int i=0; i<=100; i++){
    	System.out.println(i);
    }
```

- 상수 회: 10번이든 100번이든 1000번이든 상수 회는 O(1)이다.

```java
	for(int i=0; i<=n; i++){
    	System.out.println(i); //1
       }
       
    for(int n=0; n<=10; n++){}
     for(int i=0; i<=n; i++){
    	System.out.println(i);  //2
		}
     }
```

- n회: 1번은 n회 반복,  2번은 10n 반복한다. n이 무한대로 커지면 앞의 계수나 뒤에 붙는 상수 등은 의미가 없어진다. 그러므로 O(n)이다.

```java
    for(int n=0; n<=k; n++){
     for(int i=0; i<=n; i++){
    	System.out.println(i);  
		}
     }
```

- $n^2$회: 무한대로 도는 반복문이 중첩되어 두 번 있기 때문에 O($n^2$)이다. 이 역시 $n^2$ 의 게수나 뒤에 붙는 상수는 의미가 없다. 



<br>
Reference:
- [[Java] 자바 Queue 클래스 사용법 & 예제 총정리_코딩팩토리](https://coding-factory.tistory.com/602)
- [큐_위키백과](https://ko.wikipedia.org/wiki/%ED%81%90_(%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0))
- [스택_위키백과](https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%83%9D)
- [스택과 큐_ 포스트it의 개발자 도전기](https://seill.tistory.com/576)
- [연결 리스트_위키백과](https://ko.wikipedia.org/wiki/%EC%97%B0%EA%B2%B0_%EB%A6%AC%EC%8A%A4%ED%8A%B8)
- [[Java] 자바 LinkedList 사용법 & 예제 총정리_코딩팩토리](https://coding-factory.tistory.com/552)
- [datastructure img](https://www.computersciencedegreehub.com/faq/how-much-overlap-is-there-between-computer-science-and-data-science/)

