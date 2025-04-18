---
layout: post
title: '[JAVA] CompletableFuture 클래스'
subtitle: '비동기, 멀티쓰레드, Concurrent'
date: 2023-09-22 18:20:00 +0900
categories: [JAVA]
background: '/img/posts/etc/datastructure.jpg'
---

# CompletableFuture 클래스

- 자바 5의 Future의 한계를 이겨내고자 자바 8에 등장한 비동기 작업을 위한 클래스
- Future의 한계
> - 외부에서 완료 불가
> - 블로킹 코드인 get 으로만 이후의 결과를 처리 가능
> - 여러 Future를 조합할 수 없음, 즉 활용도가 매우 떨어짐
> - 예외 처리 불가

- Future 기반으로 외부에서 완료 시킬 수 있고(Future 인터페이스)
- 작업을 중첩시키거나 완료 후 콜백을 위해 추가할 수 있게 됨(CompletionStage 인터페이스)
> - EX) 몇 초 이내 응답 안오면 기본값 반환

<br>

## CompletableFuture 기능

### 비동기 작업 실행

- runAsync() :  리턴값 없는 경우

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    System.out.println("Thread: " + Thread.currentThread().getName());
});
future.get();
```

<br>

- supplyAsync() : 리턴값 있는 경우

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    System.out.println("Thread: " + Thread.currentThread().getName());
    return "Thread";
});
future.get();
```

- 위 두 메소드는 기본적으로 자바 7에서 나온 ForkJoinPool의 commonPool을 사용해 작업할 쓰레드를 쓰레드풀로부터 얻는다.
- 원하는 쓰레드 풀이 있다면 ExecutorService 객체를 생성해서 아래처럼 제공해주자

```java
// 쓰레드 1개인 ExecutorService를 리턴
ExecutorService executorService = Executors.newSingleThreadExecutor();
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "Thread: " + Thread.currentThread().getName();
}, executorService);

System.out.println(future.get());
System.out.println("Thread: " + Thread.currentThread().getName());

executorService.shutdown();

```

- ExecutorService : [Java - ExecutorService를 사용하는 방법](https://codechacha.com/ko/java-executors/)

<br>

### 작업 콜백

- thenApply(Function) : 반환값을 받아서 다른 값을 반환, 함수형 인터페이스 Function을 파라미터로 받음

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "thread: " + Thread.currentThread().getName();
}).thenApply(s -> {
    return s.toUpperCase();
});

System.out.println(future.get()); // 위 문자열 대문자로 리턴
```

<br>

- thenAccept(Consumer) : 반환값을 받아 처리하고 값을 반환하지 않음, 함수형 인터페이스 Consumer를 파라미터로 받음

```java
CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
    return "Thread: " + Thread.currentThread().getName();
}).thenAccept((s)->{
    System.out.println("End: " + s);
      // return 없다.
});
future.get();
```

<br>

- thenRun(Runnable) : 반환값을 받지 않고 다른 작업 실행, 함수형 인터페이스 Runnable을 파라미터로 받음

```java
CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
    return "1st Thread: " + Thread.currentThread().getName();
}).thenRun(()->{
    System.out.println("2nd Thread " + Thread.currentThread().getName());
});
future.get();
```

<br>

### 작업 조합

- thenCompose(Function) : 두 작업이 이어서 실행하도록 조합, 앞선 작업의 결과를 받아서 사용 가능, 함수형 인터페이스 Function을 파라미터로 받음

```java
public class Compose {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            return "Thread1";
        });

        CompletableFuture<String> future2 = future1.thenCompose(Compose::getCompose);
        System.out.println(future2.get());

    }
    private static CompletableFuture<String> getCompose(String message) {
        return CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            return message + " compose";
        });
    }
}
// ForkJoinPool.commonPool-worker-1
// ForkJoinPool.commonPool-worker-2
// Thread1 compose
```

<br>

- thenCombine(Function) : 두 작업을 독립적으로 실행하고, 두 작업이 완료되었을 때 콜백 실행, 함수형 인터페이스 Function을 파라미터로 받음

```java
CompletableFuture<String> thread1 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Thread1 " + Thread.currentThread().getName());
    return "Thread1";
});

CompletableFuture<String> thread2 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Thread2 " + Thread.currentThread().getName());
    return "Thread2";
});

CompletableFuture<String> result = thread1.thenCombine(thread2, (i, j) -> i + " " + j);

System.out.println(result.get());
```

<br>

- allOf : 여러 작업들을 동시에 실행하고, 모든 작업 결과에 콜백을 실행

```java
CompletableFuture<String> thread1 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Thread1 " + Thread.currentThread().getName());
    return "Thread1";
});

CompletableFuture<String> thread2 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Thread2 " + Thread.currentThread().getName());
    return "Thread2";
});

List<CompletableFuture<String>> futures = Arrays.asList(thread1, thread2);

CompletableFuture<List<String>> results =
        CompletableFuture.allOf(futures.toArray(new CompletableFuture[futures.size()]))
                .thenApply(v -> futures.stream()
                        .map(CompletableFuture::join)
                        .collect(Collectors.toList()));
results.get().forEach(System.out::println);
```

- get(), join() 는 결과가 사용 가능할 때까지 기다리고 결과가 사용 가능해지면 반환합니다.
- get() 은 checked exception을 던지고, 인터럽트가 가능하여 호출하는 스레드가 인터럽트될 때 InterruptedException을 던진다.
- join() 은 unchecked CompletionException을 던지고, 인터럽트시 CompletableFuture가 완료될 때까지 차단한다. 

<br>

- anyOf : 여러 작업들 중 가장 빨리 끝난 하나의 결과에 콜백 실행

```java
CompletableFuture<String> thread1 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Thread1 " + Thread.currentThread().getName());
    return "Thread1";
});

CompletableFuture<String> thread2 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Thread2 " + Thread.currentThread().getName());
    return "Thread2";
});

CompletableFuture<Void> future = CompletableFuture.anyOf(thread1, thread2).thenAccept(System.out::println);
future.get();
```

### 예외 처리

- exceptionally(Function) : 발생한 에러를 받아 예외로 처리, 함수형 인터페이스 Function을 파라미터로 받음

```java
@ParameterizedTest
@ValueSource(booleans =  {true, false})
void exceptionally(boolean doThrow) throws ExecutionException, InterruptedException {
    CompletableFuture<String> thread = CompletableFuture.supplyAsync(() -> {
        if (doThrow) {
            throw new IllegalArgumentException("Invalid Argument");
        }

        return "Thread: " + Thread.currentThread().getName();
    }).exceptionally(e -> {
        return e.getMessage();
    });

    System.out.println(thread.get());
}

// True:
java.lang.IllegalArgumentException: Invalid Argument
// False:
// Thread: ForkJoinPool.commonPool-worker-2
```

<br>

- handle, handleAsunc(BiFunction) : 결과값이나 에러를 받아 에러가 발생한 경우, 아닌 경우를 모두 처리할 수 있음, 함수형 인터페이스 BiFunction을 파라미터로 받음

```java
@ParameterizedTest
@ValueSource(booleans =  {true, false})
void handle(boolean doThrow) throws ExecutionException, InterruptedException {
    CompletableFuture<String> thread = CompletableFuture.supplyAsync(() -> {
        if (doThrow) {
            throw new IllegalArgumentException("Invalid Argument");
        }

        return "Thread: " + Thread.currentThread().getName();
    }).handle((result, e) -> {
        return e == null ? result : e.getMessage();
    });

    System.out.println(thread.get());
}
```

- BiFunction : 2개의 인자를 받고 1개의 객체를 리턴하는 함수형 인터페이스

```java
public interface BiFuncton<T, U, R> {
	R apply(T t, U u);
}
```

<br>

Reference:
- [[Java] CompletableFuture에 대한 이해 및 사용법 _ 망나니개발자](https://mangkyu.tistory.com/263)
- [CompletableFuture _ 고운소리의 블로그](https://gowoonsori.com/java/completablefuture/)