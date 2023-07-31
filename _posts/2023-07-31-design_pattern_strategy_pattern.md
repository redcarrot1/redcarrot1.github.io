---
title: 디자인 패턴 - 전략 패턴(Strategy Pattern)
date: 2023-07-31 12:01:00 +09:00
categories: [Design pattern]
tags:
  [
    Design pattern, Strategy Pattern
  ]
img_path: /assets/img/design_pattern/
---
## 전략 패턴(Strategy Pattern)
- 좋은 설계는 변하는 것과 변하지 않는 것을 분리하는 것이다.
- 변하지 않는 것은 `context`에, 변하는 부분은 `Strategy`라는 인터페이스로 선언하여 구현체를 생성하여 문제를 해결한다.
- 알고리즘 제품군을 정의하고 각각을 캡슐화하여 상호 교환 가능하게 만들자. 전략을 사용하면 알고리즘을 사용하는 클라이언트와 독립적으로 알고리즘을 변경할 수 있다.

### 템플릿 메소드 패턴과 비교
- 공통점: 고정된 로직과 변하는 로직이 존재. 불필요한 코드의 중복을 줄이고, 변하는 로직에 대해선 유연하게 만들기 위해 사용
- 템플릿 메소드 패턴
  - 추상 클래스에 변하지 않는 부분을, 변하는 부분을 자식 클래스에 두어서 상속(오버라이드)으로 해결
  - 추상 클래스와 자식 클래스의 의존도가 높아진다. 관계에 대한 유연성이 떨어짐
- 전략 패턴
  - 일반 클래스(Context)에 변하지 않는 부분을, 변하는 부분은 인터페이스(Strategy)에 선언 후 사용시 원하는 로직의 구현체를 만든다.
  - 인터페이스인 `Strategy`가 변하지 않는 이상, 구현체들을 변경할 필요가 없다.


## 사용 예시

![](strategy_pattern_1.png){: width="400" height="" }

### 인터페이스(Strategy)
- 변경될 로직을 선언

```java
public interface Strategy {
  void call();
}
```

### 실제 구현 클래스
- Strategy를 implements하여 변해야 하는 메소드를 구현해준다.

```java
@Slf4j
public class StrategyLogic1 implements Strategy {
  @Override
  public void call() {
    log.info("비즈니스 로직1 실행");
  }
}
```

```java
@Slf4j
public class StrategyLogic2 implements Strategy {
  @Override
  public void call() {
    log.info("비즈니스 로직2 실행");
  }
}
```

### 📌 Context
- 변하지 않는 부분의 로직을 작성
- 인터페이스인 `Strategy`를 생성자로 주입받은 후, 변해야 하는 로직을 실행해준다.

```java
@Slf4j
public class Context {
  private Strategy strategy;

  public ContextV1(Strategy strategy) {
    this.strategy = strategy;
  }

  public void execute() {
    log.info("템플릿 시작");
    
    // 변해야 하는 로직 시작
    strategy.call(); // 위임
    // 변해야 하는 로직 끝
    
    log.info("템플릿 종료");
  }
}
```

### 사용

```java
void strategyV1() {
  Strategy strategyLogic1 = new StrategyLogic1();
  ContextV1 context1 = new ContextV1(strategyLogic1);
  context1.execute();
    
  Strategy strategyLogic2 = new StrategyLogic2();
  ContextV1 context2 = new ContextV1(strategyLogic2);
  context2.execute();
}

//출력
템플릿 시작
비즈니스 로직1 실행
템플릿 종료
    
템플릿 시작
비즈니스 로직2 실행
템플릿 종료
```
- 객체 생성 시 어느 구현체를 사용하는지에 따라서 변하는 부분의 메소드가 바뀌게 된다.
- 이로써 코드 중복을 최대한 피하면서 변해야 하는 부분은 구현체 사용에 따라서 유동적으로 바꿀 수 있다.

---

## 여러가지 구현 방법

### 1. 익명 내부 클래스 사용
- 구현체를 따로 클래스를 생성하지 말고, 익명 내부 클래스로 사용시 생성할 수 있다.

```java
void strategyV2() {
  ContextV1 contextV1 = new ContextV1(new Strategy() {
    @Override
    public void call() {
      log.info("비즈니스 로직1 실행");
    }
  });
  contextV1.execute();
}
```

### 2. 람다식 사용
- 단, `Strategy` 인터페이스에 메소드가 1개만 있어야 사용 가능하다.

```java
void strategyV3() {
  ContextV1 context1 = new ContextV1(() -> log.info("비즈니스 로직1 실행"));
  context1.execute();
}
```

---

## 구현체를 메서드 인자로 전달해서 사용
- 지금까지는 `new Context( 구현체! )` 형식으로 필드에 주입해서 사용했다.
- 이번에는 `execute( 구현체! )`형식으로 직접 파라미터로 전달해서 사용해보자.

### 필드 주입 VS 인자 전달 
- 필드 주입
  -  `Context` 생성 시 선 조립. 이미 조립이 끝났기 때문에 단순히 실행만 하면 된다.
  -  구현체를 변경하려면 새롭게 `context`를 생성해야 한다는 단점.
- 인자 전달
  - 실행할 때마다 전략을 계속 지정해주어야 하지만, 유연하게 변경할 수 있다는 장점.

> 스프링에서는 이렇게 변하는 부분의 코드를 인수로 넘겨주어 사용한 패턴을 **템플릿 콜백 패턴**이라 한다.
{: .prompt-tip }


- 콜백(callback)이란 코드의 인수로서 넘겨주는 실행 가능한 코드를 말한다. (호출(call)시 코드가 뒤쪽(back)에서 실행됨)
- 스프링은 이런 패턴을 많이 사용하는데 `JdbcTemplate`, `RestTemplate`, `TransactionTemplate` 등 `XxxTemplate` 형식은 템플릿 콜백 패턴으로 만들어져 있다고 생각하면 된다.

<br>


### context 수정
- 함수 파라미터로 구현 클래스를 받을 수 있도록 만든다.

```java
@Slf4j
public class Context {

  public void execute(Strategy strategy) {
    log.info("템플릿 시작");

    // 변해야 하는 로직 시작
    strategy.call(); // 위임
    // 변해야 하는 로직 끝

    log.info("템플릿 종료");
  }
}
```

### 사용 시
- 구현체를 생성자에 넣는게 아니라, 사용 메소드에 파라미터로 넣으면 된다.

```java
void strategyV1() {
  ContextV2 context = new ContextV2();
  context.execute(new StrategyLogic1());
  context.execute(new StrategyLogic2());
}
```

- 이전과 마찬가지로 익명 내부 클래스로 사용할 수 있다.

```java
void strategyV2() {
  ContextV2 context = new ContextV2();
  context.execute(new Strategy() {
    @Override
    public void call() {
      log.info("비즈니스 로직1 실행");
    }
  });
}
```

- 람다식도 가능하다.

```java
void strategyV3() {
  ContextV2 context = new ContextV2();
  context.execute(() -> log.info("비즈니스 로직1 실행"));
}
```
