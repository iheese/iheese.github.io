---
layout: post
title: '[EFFECTIVE JAVA] ITEM 9, try-finally보다는 try-with-resources를 사용하라'
subtitle: ''
date: 2023-06-05 14:00:00 +0900
categories: [effectivejava]
background: '/img/posts/etc/datastructure.jpg'
---

# ITEM 9, try-finally보다는 try-with-resources를 사용하라

<br>

- close 메소드를 호출하여 직접 닫아줘야 하는 자바 자원 : InputStream, OutputStream, java.sql.Connection 등
- 자원 닫기는 클라이언트가 놓치기 쉬운 부분이라 안전망으로 finalpizer를 사용하고 있지만 믿을만하지 못한다.(ITEM8)

<br>

## try-finally의 단점

- 자원이 둘 이상일 경우 try-finally 방식은 너무 지저분해진다.

```java
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close(); 
            }
        } finally {
            in.close();  //중첩된 try-finally는 가독성을 떨어뜨린다. 
        }
    }
```

<br>

```java
static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
```

- 위 코드에서 예외는 try 블록, finally 블록 모두에서 일어날 수 있다.
- 물리적인 문제가 생기면 firstLineOfFile 메소드 안의 readLine 메소드가 예외 던짐 
- close 메소드 실패
- 두번째 예외가 첫번째 예외를 집어 삼키어 디버깅을 어렵게 한다.

<br>

## 해결책, try-with-resources
- AutoCloseable 인터페이스 구현 : close 메소드 정의하면 된다. 

```java
public interface AutoCloseable {
    void close() throws Exception;
} 

```

<br>

- 자원이 2개 있을 때(중첩 코드) try-with-resources를 적용한 코드

```java
    static void copy(String src, String dst) throws IOException {
        try (InputStream   in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
```

- 코드가 짧아지고, 가독성이 좋아지며 문제 진단이 쉬워진다. 
- 숨겨진 예외도 버려지지 않고, 스택 추적 내역에 '숨겨졌다(suppressed)' 는 꼬리표를 달고 출력된다.
- catch 절을 이용하여 try 문을 중첩하지 않고 다수의 예외를 처리할 수 있다.

```java
    static String firstLineOfFile(String path, String defaultVal) {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return defaultVal;
        }
    }
```

<br>

Reference:

- [EFFECTIVE JAVA](https://front.wemakeprice.com/product/121854081?search_keyword=%25EC%259D%25B4%25ED%258E%2599%25ED%258B%25B0%25EB%25B8%258C%2520%25EC%259E%2590%25EB%25B0%2594&_service=5&_no=1)
- [이펙티브 자바 3판 Github](https://github.com/WegraLee/effective-java-3e-source-code)