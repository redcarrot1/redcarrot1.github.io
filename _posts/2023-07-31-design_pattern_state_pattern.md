---
title: 디자인 패턴 - 상태 패턴(State Pattern)
date: 2023-07-31 12:02:00 +09:00
categories: [Design pattern]
tags:
  [
    Design pattern, State Pattern
  ]
img_path: /assets/img/design_pattern/
---
## 상태 패턴
- [전략 패턴](https://redcarrot1.github.io/posts/design_pattern_strategy_pattern/)과 유사한 구조를 가진다.
- 추상화한 인터페이스와 해당 인터페이스를 구현한 클래스(상태 객체)를 만들고, 컨텍스트(context)는 상태 객체에 처리를 위임하는 방식으로 구현된다.

## 예제 코드

### 상태 패턴 사용 전

```java
public class VendingMachine {
  public static enum State { NOCOIN, SELECTABLE }

  private State state = State.NOCOIN;

  public void insertCoin(int coin) {
    switch (state) {
      case NOCOIN:
        // Do Something
      case SELECTABLE:
        // Do Something
    }
  }

  public void changeState(State state) {
    this.state = state;
  }
}
```

- 자판기는 현재 상태(NOCOIN, SELECTABLE)에 따라 `insertCoin(coin)`의 로직이 달라진다.
- 만약 상태가 추가된다면 변경을 어떻게 해야할까?
  1. enum State에 새로운 상태를 추가한다.
  2. `insertCoint(coin)` 로직에 switch case를 추가한다.


> 상태를 추가하거나 로직을 변경할 때마다 코드를 변경해야 하는 것은 OCP 위반이다.
{: .prompt-warning }

<br>

### 상태 패턴 적용
![](state_pattern_1.png){: width="500" height="" }

```java
public class VendingMachine {
  private State state;

  public VendingMachine() {
    state = new NoCoinState();
  }

  public void insertCoin(int coin) {
    // 상태 객체에게 코드 구현을 위임
    state.increaseCoin(coin, this);
  }

  public void changeState(State state) {
    this.state = state;
  }
}
```

```java
public interface State {
  public void increaseCoin(int coin, VendingMachine vm);
}


public class NoCoinState implements State {
  @Override
  public void increaseCoin(int coin, VendingMachine vm) {
    // Do Something

    // Change State
    vm.changeState(new SelectableState());
  }
}
```

만약 자판기에 청소 상태 구현을 위해 State가 추가되었더라면?<br>
`CleaningState` 클래스를 **추가**하기만 하면 된다. 그럼 `VendingMachine` 클래스의 코드는 그대로 유지된다.<br>
또한 상태에 따른 로직의 변경이 일어나더라도 **해당 상태의 클래스만 변경**해주면 된다. `VendingMachine` 클래스는 건들 필요없다.<br>
물론 단점도 존재한다. **클래스 개수가 증가**하기 때문에 상태 구조를 모른다면 유지보수가 어려울 수 있다.


## 상태 변경은 누가 하는가?

```java
public class NoCoinState implements State {
  @Override
  public void increaseCoin(int coin, VendingMachine vm) {
    // Do Something

    // 컨텍스트의 상태 변경
    vm.changeState(new SelectableState());
  }
}
```

상태 객체가 `VendingMachine`을 인자로 받아 상태 변경을 하고 있다.<br>
컨텍스트의 상태를 변경할 때, 컨텍스트의 다른 값에 접근해야 할 때도 있다.<br>
따라서 컨텍스트의 메서드를 `public`으로 추가해야 한다. (예를 들면, 동전 개수에 접근하는 `getCoin()` 메서드 추가)<br>


## 전략 패턴과의 차이점

[전략 패턴](https://redcarrot1.github.io/posts/design_pattern_strategy_pattern/)과 유사하다.<br>
차이점은 객체의 상태 속성을 변환하는데 상태 패턴은 스스로 변환 할 수 있지만, 전략 패턴은 외부에서 새로운 상태의 주입이 필요하다.<br>
또한 전략 패턴은 하나의 특정 작업만 처리하는 반면, 상태 패턴은 컨텍스트 개체가 수행하는 대부분의 모든 것에 대한 기본 구현을 제공한다.

만약 `VendingMachine` 클래스를 전략 패턴으로 작성했다면 다음과 같을 것이다.

```java
public class VendingMachine {
  private State state;

  // 생성 시 상태 결정. 변경 불가 
  public VendingMachine(State state) {
    this.state = state;
  }

  public void insertCoin(int coin) {
    // 상태 객체에게 코드 구현을 위임하는건 상태 패턴과 동일
    state.increaseCoin(coin, this);
  }
}
```

또는

```java
public class VendingMachine {
    
  // 상태를 메서드 파라미터로 넘겨주기
  public void insertCoin(int coin, State state) {
    // 상태 객체에게 코드 구현을 위임하는건 동일
    state.increaseCoin(coin, this);
  }
}
```

---
## reference
개발자가 반드시 정복해야 할 객체 지향과 디자인 패턴(최범균)
