---
title: Chapter 12 - 창발성
date: 2023-07-19 12:00:00 +09:00
categories: [Book, Clean Code]
tags:
  [
    clean code,
  ]
img_path: /assets/img/clean_code/
---

## 창발적 설계로 깔끔한 코드를 구현하자
- 설계 품질 향상을 위한 단순한 설계 규칙(중요도 순)
  - 모든 테스트를 실행한다.
  - 중복을 없앤다.
  - 프로그래머 의도를 표현한다.
  - 클래스와 메서드 수를 최소로 줄인다.

### 단순한 설계 규칙 1: 모든 테스트를 실행하라
- 모든 테스트 케이스를 항상 통과하는 시스템을 '테스트가 가능한 시스템'이라 한다.
- 테스트가 가능한 시스템을 만들려면 더 나은 설계(SRP 준수 등)가 얻어진다.
- 결합도가 높으면 테스트 케이스를 작성하기 어렵다.
  - DI, 인터페이스, 추상화 등을 사용해 결합도를 낮추자.

### 단순한 2~4: 리팩터링
- 모든 테스트 케이스를 작성 후에 점진적으로 리팩터링을 진행하자
- 응집도 높이기, 결합도 낮추기, 관심사 분리, 중복 제거, 의도 표현 등을 수행한다.

## 중복을 없애라

```java
// 중복 코드 존재
public void scaleToOneDimension(float desiredDimension, float imageDimension) {
  if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
    return;
  float scalingFactor = desiredDimension / imageDimension;
  scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
  
  RenderedOpnewImage = ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor);
  image.dispose();
  System.gc();
  image = newImage;
}

public synchronized void rotate(int degrees) {
  RenderedOpnewImage = ImageUtilities.getRotatedImage(image, degrees);
  image.dispose();
  System.gc();
  image = newImage;
}
```

```java
// 공통 코드를 새 메서드로 뽑자
public void scaleToOneDimension(float desiredDimension, float imageDimension) {
  if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
    return;
  float scalingFactor = desiredDimension / imageDimension;
  scalingFactor = (float) Math.floor(scalingFactor * 10) * 0.01f);
  replaceImage(ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor));
}

public synchronized void rotate(int degrees) {
  replaceImage(ImageUtilities.getRotatedImage(image, degrees));
}

private void replaceImage(RenderedOpnewImage) {
  image.dispose();
  System.gc();
  image = newImage;
}
```
- 공통적인 코드를 새 메서드로 뽑고 보니 클래스가 SRP를 위반한다.
  - `replaceImage` 메서드를 다른 클래스로 옮겨도 좋다.

## TEMPLATE METHOD 패턴
- 좋은 설계는 변하는 것과 변하지 않는 것을 분리하는 것이다.
- 변하지 않는 것은 추상클래스의 메서드로 선언, 변하는 부분은 추상 메서드로 선언하여 자식 클래스가 오버라이딩 하도록 처리한다.
- 가장 큰 장점은 전체적으로는 동일하면서 부분적으로는 다른 구문으로 구성된 메서드의 코드 중복을 최소화시킬 수있는 점이다.

### 사용 예시
![](figure12-1.png){: width="500" height="" }

**추상 클래스**

```java
public abstract class AbstractTemplate {
    
  public void execute() {
    System.out.println("템플릿 시작");

    logic(); // 변해야 하는 로직
		
    System.out.println("템플릿 종료");
  }
    
  protected abstract void logic(); //변경 가능성이 있는 부분은 추상 메소드로 선언한다.
}
```

**실제 구현 클래스**

```java
public class SubClassLogic1 extends AbstractTemplate {
  @Override
  protected void logic() {
    System.out.println("구현 클래스 1");
  }
}
```

```java
public class SubClassLogic2 extends AbstractTemplate {
  @Override
  protected void logic() {
    System.out.println("구현 클래스 2");
  }
}
```

**사용**

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
/* 콘솔 출력
템플릿 시작
구현 클래스 1
템플릿 종료
    
템플릿 시작
구현 클래스 2
템플릿 종료
*/
```

- 객체 생성 시 어느 구현체를 사용하는지에 따라서 변하는 부분의 메소드가 바뀌게 된다.
- 이로써 코드 중복을 최대한 피하면서 변해야 하는 부분은 구현체 사용에 따라서 유동적으로 바꿀 수 있다.

**익명 내부 클래스 사용**
```java
public class templateMethod1 extends AbstractTemplate {
  public static void main(String[] args) {
	AbstractTemplate template1 = new AbstractTemplate() {
		@Override
		protected void logic() {
			System.out.println("구현 클래스 1");
		}
	};
	template1.execute();
    
	AbstractTemplate template2 = new AbstractTemplate() {
		@Override
		protected void logic() {
			System.out.println("구현 클래스 2");
		}
	};
	template2.execute();
  }
}
```

## 표현하라
1. 좋은 이름을 선택한다.
2. 함수와 클래스 크기를 가능한 줄인다.
  - 이름짓기도, 구현하기도, 이해하기도 쉬워진다.
3. 표준 명칭을 사용한다.
  - 예를 들어 디자인 패턴은 사용 시 클래스 이름에 패턴 이름을 넣어준다.
4. 단위 테스트 케이스를 꼼꼼이 작성한다.

## 클래스와 메서드 수를 최소로 줄여라
- 무의미하고 독단적인 견해는 멀리하고 실용적인 방식을 택하자
  - 예를 들어, 클래스마다 무조건 인터페이스를 생성하라는 주장, 자료 클래스와 동작 클래스를 무조건 분리하라는 주장
- 이 규칙은 우선순위가 가장 낮다.
  - 클래스와 메서드 수가 많아지더라도, 테스크 케이스를 만들고 중복을 제거하고 의도를 표현하는 작업이 더 중요하다.

> 경험을 대신할 단순한 개발 기법은 없다.