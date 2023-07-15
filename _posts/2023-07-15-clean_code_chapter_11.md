---
title: Chapter 11 - 시스템
date: 2023-07-15 12:00:00 +09:00
categories: [Book, Clean Code]
tags:
  [
    clean code,
  ]
img_path: /assets/img/clean_code/
---

## 시스템 제작과 시스템 사용을 분리하라
> 소프트웨어 시스템은 (애플리케이션 객체를 제작하고 의존성을 서로 '연결'하는) 준비 과정과 (준비 과정 이후에 이어지는) 런타임 로직을 분리해야 한다.

```java
public Service getService() {
    if (service == null)
        service = new MyServiceImpl(...);
    return service;
}
```

- 위의 코드를 초기화 지연(Lazy Initialization) 혹은 계산 지연(Lazy Evaluation)이라는 기법이다.
  - 장점
    - 실제로 필요할 때까지 객체를 생성하지 않으므로 불필요한 부하가 걸리지 않는다.
    - 어떤 경우에도 null 포인터를 반환하지 않는다.
  - 단점
    - `MyServiceImpl`의 생성자에 명시적으로 의존한다.(실제 런타임 시 사용하지 않더라도 의존성 해결이 안되면 컴파일이 안된다.)
    - `service`가 null인 경우와 아닌 경우, 두 방식을 모두 테스트해야한다.
    - `MyServiceImpl`이 모든 상황에 적합한 객체인지 모른다.

**Main 분리**
- 생성과 관련한 코드는 모두 main이나 main이 호출하는 모듈로 옮기고, 나머지 시스템은 모든 객체가 생성되었고 모든 의존성이 연결되었다고 가정한다.

![](figure 11-1.png)

- 의존성 화살표 방향에 주목하자. 모든 화살표가 main 쪽에서 애플리케이션 쪽을 향한다.
- 즉, 애플리케이션은 main이나 객체가 생성되는 과정을 전혀 모른다는 뜻이다.
- 단지 모든 객체가 적절히 생성되었다고 가정한다.

**팩토리 기법**
- 때로는 객체가 생성되는 시점을 애플리케이션이 결정할 필요도 생긴다.
- 예를 들어, (아래 이미지에서는) LineItem을 생성하는 시점은 `OrderProcessing`가 결정하지만, 구체적인 코드는 알 수 없다.
- 마찬가지로 의존성 화살표 방향이 main에서 애플리케이션 쪽으로 향한다.

![](figure 11-2.png)

**의존성 주입(DI)**
- 의존성 주입은 제어 역전(IoC) 기법을 의존성 관리에 적용한 메커니즘이다.
- 객체는 의존성 자체를 인스턴스로 만드는 책임을 지는 대신, main루틴이나 특수 컨테이너에게 맡긴다.
  - 의존성을 주입하는 방법으로 setter 메서드나 생성자 인수를 제공한다.
  - DI 컨테이너는 필요한 객체의 인스턴스를 만든 후 필요한 클래스에게 의존성을 설정한다.

```java
// Spring framwork는 자바 DI 컨테이너를 제공한다.
@Service
@RequiredArgsConstructor
public class AdminPostService {
  
  private final PostRepository postRepository;

  public AdminPostService(PostRepository postRepository) {
    this.postRepository = postRepository; // 의존성 주입
  }
  ...
}
```

## 확장
- 작은 마을이 성장하며 도로를 확장 공사할 때 "왜 처음부터 넓게 만들지 않았지?"하는 생각이 들 수 있다.
  - 성장할지 모른다는 기대로 처음부터 거대하게 공사하는게 올바르다고 정당화할 수 있는가?
- '처음부터 올바르게' 시스템을 만들 수 있다는 믿음은 미신이다.
  - 새로운 스토리가 생기면 거기에 맞춰 시스템을 조정하고 확장하면 된다.
- TDD, 리팩터링, Clean code는 코드 수준에서 시스템을 조정하고 확장하기 쉽게 만든다.

## 프록시
- Client에서 Server을 직접 호출하고, 처리 결과를 직접 받는다. 이것을 **직접 호출**이라 한다.
  - Client -> Server
- Client에서 Server을 직접 호출하는 것이 아니라 어떤 대리자를 통해서 대신 간접적으로 서버에 요청할 수도 있다. 이것을 **간접 호출**이라 한다.
  - 여기서 간접 호출하는 대상을 **프록시(Proxy)**라 한다.
    - Client -> Proxy -> Server

- 프록시는 Client와 Server의 중간에 있기 때문에 여러가지 일을 수행 할 수 있다.
  - 권한에 따른 접근 차단, 캐싱, 지연로딩을 수행하는 **접근 제어**
  - 서버의 기능에 다른 기능까지 추가해주는 **부가 기능 추가** (요청, 응답값을 변형해주거나 로그 기록 등)
  - 대리자가 또 다른 대리자를 호출하는 **프록시 체인**

### 프록시는 대체 가능해야 한다.
- 아무 객체나 프록시가 되는것은 아니다.
- 클라이언트 입장에서는 서버에 요청을 한건지, 프록시에게 요청을 한 것인지 조차 몰라야 한다.
- 즉, 서버와 프록시는 **같은 인터페이스**를 사용해야 하며, 클라이언트의 코드를 하나도 건들이지 않고 **프록시 추가와 런타임 객체 의존 관계 주입**만 변경해서 사용할 수 있어야 한다.
- Client는 ServerInterface를 의존해야 한다. 그리고 ServerInterface의 구현체로 실제 서버와 Proxy가 존재한다.
따라서 DI를 사용해서 대체 가능하다.

> JDK 동적 프록시는 인터페이스를 기반으로 프록시를 동적으로 만들어준다. 따라서 인터페이스가 필수이다.
{: .prompt-warning }

```java
public interface AInterface {
  String call();
}
```

```java
public class AImpl implements AInterface {
  public String call() {
    System.out.println("A 호출");
    return "a";
  }
}
```

```java
public class TimeInvocationHandler implements InvocationHandler {
  private final Object target;

  public TimeInvocationHandler(Object target) {
    this.target = target;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    System.out.println("TimeProxy 실행");
    long startTime = System.currentTimeMillis();

    Object result = method.invoke(target, args);


    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    System.out.println("TimeProxy 종료 resultTime = " + resultTime);
    return result;
  }
}
```

```java
public class Main {
  public static void main(String[] args) {
    AInterface target = new AImpl();
    TimeInvocationHandler handler = new TimeInvocationHandler(target);

    AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler);
    proxy.call();
  }
}
```
```text
// 실행 결과
TimeProxy 실행
A 호출
TimeProxy 종료 resultTime = 0
```

## 테스트 주도 시스템 아키텍처 구축
- 소프트웨어 프로젝트에서는 BDUF(Big Design Up Front)는 해롭기까지 한다.
  - 처음 선택한 아키텍처의 사고 방식 때문에 변경을 쉽사리 수용하지 못하기 때문이다.
- 아주 단순하면서 잘 분리된 아키텍처로 결과물을 빠르게 출신 후, 기반 구조를 추가하며 조금씩 확장해 나가도 괜찮다.
