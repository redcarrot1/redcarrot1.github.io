---
title: Chapter 06 - 객체와 자료 구조
date: 2023-07-10 12:00:00 +09:00
categories: [Book, Clean Code]
tags:
  [
    clean code,
  ]
---

- 변수를 비공개(`private`)로 정의하는 이유가 있다. 남들이 변수에 의존하지 않게 만들고 싶어서다.
- 그런데 많은 프로그래머는 조회(`get`) 함수와 설정(`set`) 함수를 당연하게 공개(`public`)해 변수를 외부에 노출할까?

## 자료 추상화
- 인터페이스나 getter/setter 함수만으로는 추상화가 이루어지지 않는다.
- 아무 생각없이 getter/setter 함수를 추가하는 방법이 가장 나쁘다.

```java
// Bad(구체적인 Point 클래스)
// 확실히 직교좌표계를 사용한다.
// 변수가 private 이라도, getter/setter를 제공한다면 구현을 외부로 노출하는 셈이다.
public class Point { 
  public double x; 
  public double y;
}
```

```java
// Good(추상적인 Point 클래스)
// 점이 직교좌표계인지 극좌표계인지 알 수 없다.
// 구현을 감추려면 추상화가 필요하다.
public interface Point {
  double getX();
  double getY();
  void setCartesian(double x, double y); 
  double getR();
  double getTheta();
  void setPolar(double r, double theta); 
}
```

<br>

```java
// Bad
// 두 함수가 변수값을 읽어 반환할 뿐이라는 사실이 거의 학실하다.
public interface Vehicle {
	double getFuelTankCapacityInGallons();
	double getGallonsOfGasoline();
}
```

```java
// Good
// 자동차 연료 상태를 백분율이라는 추상적인 개념으로 알려준다.
public interface Vehicle {
	double getPercentFuelRemaining();
}
```

## 자료/객체 비대칭
- 객체 : 추상화 뒤로 자료를 숨긴 채, 자료를 다루는 함수만 공개한다.
- 자료 구조 : 자료를 그대로 공개하며, 별다른 함수는 제공하지 않는다.

```java
// 절차적인 도형
// Square, Rectangle, Circle은 자료구조이다.(아무 메서드도 제공하지 않는다.)
// 도형이 동작하는 방식은 Geometry 클래스에서 구현한다.
// 새로운 함수를 추가하고 싶다면 Geometry 클래스만 수정하면 된다.
// 반면 새로운 도형을 추가하고 싶다면 Geometry 클래스의 모든 함수를 고쳐야 한다.

public class Square { 
  public Point topLeft; 
  public double side;
}

public class Rectangle { 
  public Point topLeft; 
  public double height; 
  public double width;
}

public class Circle { 
  public Point center; 
  public double radius;
}

public class Geometry {
  public final double PI = 3.141592653589793;
  
  public double area(Object shape) throws NoSuchShapeException {
    if (shape instanceof Square) { 
      Square s = (Square)shape; 
      return s.side * s.side;
    } else if (shape instanceof Rectangle) { 
      Rectangle r = (Rectangle)shape; 
      return r.height * r.width;
    } else if (shape instanceof Circle) {
      Circle c = (Circle)shape;
      return PI * c.radius * c.radius; 
    }
    throw new NoSuchShapeException(); 
  }
}
```

```java
// 객체 지향적인 도형
// 여기서 area()는 다형메서드다.
// 새 도형을 추가해도 기존 함수에 아무런 영향을 미치지 않는다.
// 반면 새 함수를 추가하고 싶다면 도형 클래스 전부를 고쳐야 한다.

public class Square implements Shape { 
  private Point topLeft;
  private double side;
  
  public double area() { 
    return side * side;
  } 
}

public class Rectangle implements Shape { 
  private Point topLeft;
  private double height;
  private double width;

  public double area() { 
    return height * width;
  } 
}

public class Circle implements Shape { 
  private Point center;
  private double radius;
  public final double PI = 3.141592653589793;

  public double area() {
    return PI * radius * radius;
  } 
}
```

> (자료 구조를 사용하는) 절차적인 코드는 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다. 반면, 객체 지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다.

> 절차적인 코드는 새로운 자료 구조를 추가하기 어렵다. 그러려면 모든 함수를 고쳐야 한다. 객체 지향 코드는 새로운 함수를 추가하기 어렵다. 그러려면 모든 클래스를 고쳐야 한다.

- 새로운 자료 타입이 필요한 경우 : 클래스와 객체 지향 기법이 적합
- 새로운 함수가 필요 : 자료 구조와 절차적인 코드가 적합

## 디미터 법칙
- 클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다.
	- 클래스 C
	- C 인스턴스 변수에 저장된 객체
	- f가 생성한 객체
	- f 인수로 넘어온 객체

### 기차 충돌
- `final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();`
	- 위와 같은 코드를 기차 충돌(train wreck)라 부른다.
	- 일반적으로 조잡하다 여겨지는 방식이므로 피하는 편이 좋다.
	- 따라서 다음과 같이 나누는 편이 좋다.
	```java
	Options opts = ctxt.getOptions();
	File scratchDir = opts.getScratchDir();
	final String outputDir = scratchDir.getAbsolutePath();
	```
- 위 예제는 디미터 법칙을 위반할까?
	- `ctxt`, `Options`, `ScratchDir`이 객체이면 위반이다.
	- 반면, 자료구조라면 단연히 내부 구조를 노출하므로 위반이 아니다.
- 위 예제는 `getter` 함수를 사용하는 바람에 혼란을 일으킨다.

### 잡종 구조
- 절반은 객체, 절반은 자료 구조인 잡종 구조는 중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 getter/setter 함수도 있다.
- 이런 잡종 구조는 새로운 함수와 새로운 자료 구조를 추가하기 어렵다.(양쪽에서 단점만 모아놓은 구조)
- 그러므로 잡종 구조는 되도록 피하는 편이 좋다.
- 프로그래머가 함수나 타입을 보호할지 공개할지 확신하지 못해 (더 나쁘게는 무지해) 어중간하게 내놓은 설계에 불과하다.

### 구조체 감추기
- `final String outputDir`가 어디에 쓰이는지 아래에 보니 다음과 같은 코드가 있다고 가정하자

```java
String outFile = outputDir + "/" + className.replace('.', '/') + ".class"; 
FileOutputStream fout = new FileOutputStream(outFile); 
BufferedOutputStream bos = new BufferedOutputStream(fout);
```
- 위 코드에 따르면 경로를 얻으려는 이유가 임시 파일을 생성하기 위함을 알 수 있다.
- 그렇다면 ctxt 객체에 임시 파일을 생성하라고 시키면 어떨까?

```java
// Good
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

## 자료 전달 객체
- 자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스다.
- 때로는 DTO(Data Transfer Object)라 한다.
- 좀 더 일반적인 형태는 빈(bean) 구조이다.
	- 빈은 `private` 함수를 getter/setter로 조작한다.(일종의 사이비 캡슐화..)

### 활성 레코드
- 활성 레코드는 DTO의 특수한 형태다.
- private 변수 + getter/setter + 탐색 함수(save, find ..)
- 활성 레코드에 비즈니스 로직을 추가해서 객체로 취급하지 말자
	- 활성 레코드의 자료를 사용하면서 비즈니스 규칙을 담는 객체는 따로 생성한다.

## 결론
- 객체는 동작을 공개하고 자료를 숨긴다.
	- 기존 동작으로 변경하지 않으면서 새 객체 타입을 추가하기는 쉽다
	- 기존 객체에 새 동작을 추가하기는 어렵다.
- 자료구조는 별다른 동작 없이 자료를 노출한다.
	- 새 동작을 추가하기는 쉽다.
	- 기존 함수에 새 자료 구조를 추가하기는 어렵다.