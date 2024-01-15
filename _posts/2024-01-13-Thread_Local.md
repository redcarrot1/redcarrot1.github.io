---
title: Java - ThreadLocal
date: 2024-01-13 10:00:00 +09:00
categories: [Spring-boot]
tags:
  [
    Java, ThreadLocal
  ]
img_path: /assets/img/etc/thread_local/
---

멀티 쓰레드 프로그램에서 동시성(concurrency) 문제는 언제나 고민거리이다. 아래는 동시성 문제가 발생하는 예시이다.

```java
public class Singleton {

    public static String value;

    private Singleton() {
    }

    public static void store(String value) {
        String threadName = "[" + Thread.currentThread().getName() + "] ";
        System.out.println(threadName + "try: " + Singleton.value + " -> " + value);

        Singleton.value = value;
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
        }

        System.out.println(threadName + "조회: " + Singleton.value);
    }

    public static void main(String[] args) {
        Runnable runnable1 = () -> {
            Singleton.store("userA");
        };
        Runnable runnable2 = () -> {
            Singleton.store("userB");
        };

        Thread thread1 = new Thread(runnable1);
        thread1.setName("Thread-A");
        Thread thread2 = new Thread(runnable2);
        thread2.setName("Thread-B");

        thread1.start();
        thread2.start();
    }
}
/* 결과
[Thread-A] try: null -> userA
[Thread-B] try: null -> userB
[Thread-B] 조회: userB
[Thread-A] 조회: userB
*/
```

동시성 문제는 상황에 따라 발생 여부가 달라진다. OS, JVM 등의 쓰레드 스케줄링 정책에 따라서 결과가 달라진다.<br>
따라서 테스트 코드만으로 문제 발생 여부를 판단하기는 역부족이다. 개발자가 관련 지식을 미리 알고 대처해야한다.<br>
특히 스프링은 싱글톤 빈을 사용하기 때문에 동시성 문제에 더 취약하다.

critical section을 보호하기 위해 (컴공이면 지겹게 들었을) mutex, semaphore이 존재한다.<br>
자바에서는 동기화 블록을 위한 `synchronized` 키워드가 있다. `synchronized`는 인스턴스에 lock을 걸 수 있고, 메서드 선언부에도 사용할 수 있다.<br>
이번 게시물에서는 쓰레드마다 특별한 저장소를 제공하는 `ThreadLocal`에 대해 알아보려고 한다.

## ThreadLocal
쓰레드 로컬은 해당 쓰레드만 접근할 수 있는 특별한 저장소를 말한다. 다시말해 쓰레드마다 별도의 내부 저장소를 제공한다.<br>
쓰레드 로컬은 멀티쓰레드 디자인 패턴 중 하나이다. 자바에서는 특별하게 `java.lang.ThreadLocal` 클래스를 제공해준다.

![](1.png)

```java
public class Singleton {

    public static ThreadLocal<String> value = new ThreadLocal<>();

    private Singleton() {
    }

    public static void store(String value) {
        String threadName = "[" + Thread.currentThread().getName() + "] ";
        System.out.println(threadName + "try: " + Singleton.value.get() + " -> " + value);

        Singleton.value.set(value);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
        }

        System.out.println(threadName + "조회: " + Singleton.value.get());
    }

    public static void remove() {
        Singleton.value.remove();
    }

    public static void main(String[] args) {
        Runnable runnable1 = () -> {
            Singleton.store("userA");
            Singleton.remove();
        };
        Runnable runnable2 = () -> {
            Singleton.store("userB");
            Singleton.remove();
        };

        Thread thread1 = new Thread(runnable1);
        thread1.setName("Thread-A");
        Thread thread2 = new Thread(runnable2);
        thread2.setName("Thread-B");

        thread1.start();
        thread2.start();
    }
}
/* 결과
[Thread-A] try: null -> userA
[Thread-B] try: null -> userB
[Thread-A] 조회: userA
[Thread-B] 조회: userB
*/
```

사용 방법은 간단하다. 먼저 쓰레드 로컬로 관리하려는 객체를 `ThreadLocal<>`로 감싼 타입으로 선언해준다.<br>
이후 값을 저장할 땐 `ThreadLocal.set(xx)`, 값을 조회할 땐 `ThreadLocal.get()`을 사용하면 된다.<br>

### 주의할 점
쓰레드 풀을 사용할 때는 쓰레드 로컬 값을 remove 하는 과정이 필요하다. 쓰레드 풀은 쓰레드가 삭제되는게 아니므로 쓰레드 로컬도 그대로 남아있다.<br>
따라서 재사용되는 쓰레드인 경우 기존에 저장되어 있던 쓰레드 로컬의 값을 조회할 가능성이 생긴다.<br>
따라서 쓰레드 로컬 사용이 끝나면 `ThreadLocal.remove()`을 이용해 저장된 값을 제거해주자.

![](2.png)
![](3.png)

## Spring Security에서 활용
Spring Security에서 ContextHolder가 ThreadLocal로 구현되어 있다.
```java
final class ThreadLocalSecurityContextHolderStrategy implements SecurityContextHolderStrategy {
    private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<>();
    ... (생략)
}
```

보통 ContextHolder의 내부에는 사용자 정보인 `Authentication` 객체를 담게 되는데, 쓰레드 로컬을 사용함으로써 여러 Thread의 요청에도 꼬이지 않게 해준다.

## Reference
[김영한 스프링 핵심 원리 - 고급편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8)