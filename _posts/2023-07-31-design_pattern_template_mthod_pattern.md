---
title: 디자인 패턴 - 템플릿 메소드 패턴(Template Method Pattern)
date: 2023-07-31 12:00:00 +09:00
categories: [Design pattern]
tags:
  [
    Design pattern, Template Method Pattern
  ]
img_path: /assets/img/design_pattern/
---

## 템플릿 메소드 패턴
- 좋은 설계는 변하는 것과 변하지 않는 것을 분리하는 것이다.
- 변하지 않는 것은 추상클래스의 메서드로 선언, 변하는 부분은 추상 메서드로 선언하여 자식 클래스가 오버라이딩 하도록 처리한다.
- 이렇듯이 특정 작업을 처리하는 일부분을 서브 클래스로 캡슐화하여 전체적인 구조는 바꾸지 않으면서 특정 단계에서 수행하는 내용을 바꾸는 패턴이다.

가장 큰 장점은 전체적으로는 동일하면서 부분적으로는 다른 구문으로 구성된 메서드의 **코드 중복을 최소화**시킬 수있는 점이다.

## 사용 예시
![](template_method_pattern_1.png){: width="300" height="" }

### 추상 클래스

```java
public abstract class AbstractTemplate {
    
  public void execute() {
    System.out.println("템플릿 시작");
  
    // 변해야 하는 로직 시작
    logic(); // Do Something
    // 변해야 하는 로직 종료
  
    System.out.println("템플릿 종료");
  }
  
  // 변경 가능성이 있는 부분은 추상 메소드로 선언한다.
  protected abstract void logic();
}
```

- 추상 클래스에서 변경 가능성이 있는 부분은 추상 메소드로 작성한다.

### 실제 구현 클래스
- 추상클래스를 extends하여 변해야 하는 메소드를 Override한다.

```java
public class SubClassLogic1 extends AbstractTemplate {
  @Override
  protected void logic() {
    System.out.println("변해야 하는 메서드는 이렇게 오버라이딩으로 사용1.");
  }
}
```

```java
public class SubClassLogic2 extends AbstractTemplate {
  @Override
  protected void logic() {
    System.out.println("변해야 하는 메서드는 이렇게 오버라이딩으로 사용2.");
  }
}
```

### 사용

```java
public class templateMethod1 extends AbstractTemplate {
  public static void main(String[] args) {
    AbstractTemplate template1 = new SubClassLogic1();
    template1.execute();
  
    System.out.println();
    
    AbstractTemplate template2 = new SubClassLogic2();
    template2.execute();
  }
}

//출력
템플릿 시작
변해야 하는 메서드는 이렇게 오버라이딩으로 사용1.
템플릿 종료
    
템플릿 시작
변해야 하는 메서드는 이렇게 오버라이딩으로 사용2.
템플릿 종료
```

- 객체 생성 시 어느 구현체를 사용하는지에 따라서 변하는 부분의 메소드가 바뀌게 된다.
- 이로써 코드 중복을 최대한 피하면서 변해야 하는 부분은 구현체 사용에 따라서 유동적으로 바꿀 수 있다.



## 익명 내부 클래스 이용
- `SubClassLogic1`, `SubClassLogic2`처럼 구현 클래스를 계속 만들어야 하는 단점이 있다.
- **해결 방법: 익명 내부 클래스를 사용**
- 익명 내부 클래스를 사용하면 객체 인스턴스를 생성하면서 동시에 생성할 클래스를 상속 받은 자식 클래스를 정의할 수 있다.

```java
public class templateMethod1 extends AbstractTemplate {
  public static void main(String[] args) {
    AbstractTemplate template1 = new AbstractTemplate() {
      @Override
      protected void logic() {
        System.out.println("익명 내부 클래스로 구현 1");
      }
    };
    template1.execute();
      
    AbstractTemplate template2 = new AbstractTemplate() {
      @Override
      protected void logic() {
        System.out.println("익명 내부 클래스로 구현 2");
      }
    };
    template2.execute();
  }
}

//출력
템플릿 시작
익명 내부 클래스로 구현 1
템플릿 종료
    
템플릿 시작
익명 내부 클래스로 구현 2
템플릿 종료
```