---
title: Java - Dynamic proxy
date: 2024-01-15 10:00:00 +09:00
categories: [Java]
tags:
  [
    Java, Dynamic proxy, CGLIB
  ]
img_path: /assets/img/etc/dynamic_proxy/
---

프록시 패턴이나 데코레이터 패턴을 사용하면 프록시 클래스를 손쉽게 만들 수 있다. 하지만 필요한 프록시 클래스 개수가 증가하면, 디자인 패턴을 구현할 클래스도 증가하게 된다.<br>
본 게시물에서는 이러한 상황에서 프록시를 공통 추상화시킬 수 있는 동적 프록시(Dynamic proxy)를 알아보려고 한다.

## 선행 : 리플렉션
JDK 동적 프록시를 이해하기 위해서는 자바의 리플렉션 기술을 알아야한다. 리플렉션은 클래스나 메서드의 메타정보를 **동적으로** 획득하고 호출할 수 있다.<br>리플렉션의 자세한 설명은 다른 게시물을 참고하고, 본편에서는 최소한의 기능정도를 보려고한다.

```java
@Slf4j
public class ReflectionTest {
    @Test
    void reflection() throws Exception {
        Class classHello = Class.forName("org.example.springtest.proxy.jdkdynamic.ReflectionTest$Hello");
        Hello target = new Hello();

        Method methodCallA = classHello.getMethod("callA");
        dynamicCall(methodCallA, target);

        Method methodCallB = classHello.getMethod("callB");
        dynamicCall(methodCallB, target);
    }

    private void dynamicCall(Method method, Object target) throws Exception {
        log.info("start");
        Object result = method.invoke(target);
        log.info("result={}", result);
    }


    @Slf4j
    static class Hello {
        public String callA() {
            log.info("callA");
            return "A";
        }

        public String callB() {
            log.info("callB");
            return "B";
        }
    }
}
```
`Class.forName()`을 이용하면 클래스 메타정보를 얻을 수 있다. 이때 인자로는 클래스의 패키지 경로까지 모두 작성해준다. 내부 클래스는 구분을 위해 `$`을 사용한다.<br>
`classHello.getMethod()`를 이용하면 메서드 메타정보를 얻을 수 있다. 그리고 `method.invoke(target)`을 수행하면 인자에 있는 `target` 인스턴스의 `method`를 호출할 수 있다.

리플렉션은 애플리케이션을 동적으로 유연하게 만들어주지만 **컴파일 시점에 에러를 잡을 수 없다는 단점**이 있다. 따라서 리플렉션은 일반적으로 사용하면 안되고, 매우 일반적인 공통 처리가 필요할 때 주의하며 사용해야한다.<br>
예로 스프링은 리플렉션을 내부에서 적절하게 사용하고 있다. `@RequestBody`가 붙으면 역직렬화를 수행하거나 의존성 주입(DI)도 리플렉션을 사용한다.<br><br>

참고) 사실 위 코드는 리플렉션말고 람다로도 해결이 가능하다.
```java
@Test
void lambdaTest() {
    Hello target = new Hello();
    lambda(target::callA);
    lambda(target::callB);
}

private String lambda(Supplier<String> supplier) {
    log.info("start");
    String result = supplier.get();
    log.info("result={}", result);
    return result;
}
```

<br>

## JDK 동적 프록시
JDK 동적 프록시에 적용할 로직은 `InvocationHandler` 인터페이스를 구현해서 작성하면 된다.
```java
package java.lang.reflect;

public interface InvocationHandler {
    // proxy : 프록시 자신, method : 호출할 메서드, args : 메서드를 호출할 때 전달한 인수
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```

메서드 실행 시간을 측정하는 프록시 예제를 살펴보자.
```java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

    private final Object target; // 동적 프록시가 호출할 인스턴스

    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log.info("Time Proxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = method.invoke(target, args); // 리플렉션을 사용해 메서드 호출

        long endTime = System.currentTimeMillis();
        log.info("Time Proxy 종료, resultTime={}", endTime - startTime);
        return result;
    }
}
```

JDK 동적 프록시를 사용하기 위해서는 실행할 클래스의 인터페이스가 필수로 필요하다.(코드 생략)<br>
프록시를 동적 생성 후 호출을 위해선 아래 코드를 사용하면 된다.
```java
InvocationHandler handler = new TimeInvocationHandler(new AImpl());

AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[] {AInterface.class}, handler);

proxy.call();
```
먼저 프록시를 적용할 구현체를 인자로 가진 `InvocationHandler`를 만든다.<br>
`Proxy.newProxyInstance()` 메서드로 프록시 객체(Object)를 반환받을 수 있는데, 적절하게 casting해주자. 인자로는 클래스 로더 정보, 인터페이스, InvocationHandler를 넣어주면 된다.<br>
마지막으로 생성된 프록시를 사용하면 된다.(예시코드에서 AInterface에는 `call()` 이란 메서드가 존재한다.)
![](1.png)

<br>

## CGLIB
CGLIB는 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리이다. 스프링에 기본적으로 포함되어 있다.<br>
(JDK 동적 프록시의 단점인) 인터페이스 없이도 동적 프록시를 만들 수 있다는게 특징이다.<br>
CGLIB를 실행하기 위해 먼저 `MethodInterceptor`를 구현해야 한다.

```java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {
    private final Object target;

    public TimeMethodInterceptor(Object target) {
        this.target = target;
    }

    // obj: CGLIB가 적용된 객체, method: 호출된 메서드, args: 메서드와 함께 전달된 인수, methodProxy: 메서드 호출에 사용
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        // Method를 이용해 호출해도 되지만, 내부 최적화로 인해 methodProxy를 사용하는게 좋다.
        Object result = methodProxy.invoke(target, args); // 프록시 객체 실행

        long endTime = System.currentTimeMillis();
        log.info("TimeProxy 종료 resultTime={}", endTime-startTime);
        return result;
    }
}
```

CGLIB를 사용하기 위해서는 `Enhancer` 클래스가 필요하다. 이 클래스에 target 클래스 타입과 `MethodInterceptor`을 넣어줘야 한다.<br>
이후 `create()`을 호출하면 프록시가 적용된 객체를 얻을 수 있다.
```java
void cglib() {
    ConcreteService target = new ConcreteService();

    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(ConcreteService.class);
    enhancer.setCallback(new TimeMethodInterceptor(target));
    ConcreteService proxy = (ConcreteService) enhancer.create();
    proxy.call();
}
```

사실 우리가 직접 CGLIB를 직접 사용할 일은 거의 없다.(생성자 제약 등)<br>
스프링 부트는 AOP를 적용할 때 기본적으로 항상 CGLIB를 사용해서 프록시를 생성한다. 때문에 전체적인 컨셉만 알고있으면 될 것 같다.

## Reference
[김영한 스프링 핵심 원리 - 고급편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8)