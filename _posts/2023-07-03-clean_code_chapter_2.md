---
title: Chapter 02 - 의미 있는 이름
date: 2023-07-03 12:00:00 +09:00
categories: [Book, Clean Code]
tags:
  [
    clean code,
  ]
img_path: /assets/img/etc/aws_hacking/
---
### 의도를 분명히 밝혀라
- 변수, 함수, 클래스는 존재 이유, 수행 기능, 사용 방법을 주석 없이 이름만으로도 파악할 수 있어야 한다.
- 예시1: 이름 `d`는 아무 의미도 드러나지 않는다. 주석없이, 측정하려는 값과 단위를 표현하는 이름이 필요하다.
  ```java
  // Bad
  int d; // 경과 시간(단위: 날짜)

  // Good
  int elapsedTimeInDays;
  int daysSinceCreation;
  int daysSinceModification;
  int fileAgeInDays;
  ```
- 예시2
  - Bad: `theList`에 무엇이 들어있는지, 0번째 인덱스는 왜 중요한지, 값 4는 무슨 의미인지 등을 알 수 없다. 

  ```java
  // Bad
  List<int[]> theList;

  public List<int[]> getThem() {
      List<int[]> list1 = new ArrayList<int[]>();
      for (int[] x : theList)
          if (x[0] == 4)
              list1.add(x);
      return list1;
  }
  ```
  - 개선1: 변수와 값이 어떤 의미를 갖는지 파악하도록 이름을 지은다.

  ```java
  List<int[]> gameBoard;
  static final int STATUS_VALUE = 0;
  static final int FLAGGED = 4;

  public List<int[]> getFlaggedCells() {
      List<int[]> flaggedCells = new ArrayList<>();
      for (int[] cell : gameBoard)
          if (cell[STATUS_VALUE] == FLAGGED)
              flaggedCells.add(cell);
      return flaggedCells;
  }
  ```
  - 개선2: `int[]` 대신 클래스로 만들고, `FLAGGED` 상수를 감추는 대신 명시적인 함수를 사용하자

  ```java
  public class Cell {
      private static final int STATUS_VALUE = 0;
      private static final int FLAGGED = 4;
      private int[] value;

      public boolean isFlagged() {
          return value[STATUS_VALUE] == FLAGGED;
      }
  }


  List<Cell> gameBoard;

  public List<Cell> getFlaggedCells() {
      List<Cell> flaggedCells = new ArrayList<>();
      for (Cell cell : gameBoard)
          if (cell.isFlagged())
              flaggedCells.add(cell);
      return flaggedCells;
  }
  ```

### 그릇된 정보를 피하라
- 나름대로 널리 쓰이는 의미가 있는 단어를 다른 의미로 사용하지 마라
  - 예를 들어, hp, aix, sco는 유닉스 플랫폼이나 변종을 가리키는 이름이기 때문에 변수 이름으로 적합하지 않다.
- `List` 등 컨테이너 유형을 이름에 넣지 마라
- 서로 흡사한 이름을 사용하지 않도록 주의하라
- 유사한 개념은 유사한 표기법을 사용하라
  - 특히 IDE의 코드 자동 완성 기능을 사용하기 위해(물론 각 개념의 차이는 명백히 드러나야 한다.)
- 비슷해 보이는 문자를 주의하라
  - l vs 1 (소문자 L과 숫자 1)
  - O vs 0 (대문자 O와 숫자 0)

### 의미 있게 구분하라
- 아무런 정보를 제공하지 않는 이름이나 저자 의도가 전혀 드러나지 않는 이름을 사용하지 마라

  ```java
  // Bad
  public static void copyChars(char[] a1, char[] a2) {
      for (int i = 0; i < a1.length; i++) {
          a2[i] = a1[i];
      }
  }
  ```
  ```java
  // Good
  public static void copyChars(char[] source, char[] destination) {
      for (int i = 0; i < source.length; i++) {
          destination[i] = source[i];
      }
  }
  ```
- 불용어(noise word)를 추가한 이름은 아무 정보도 제공하지 못한다.
  - 불용어 예시: info, data, a, an, the, 변수타입
  - Product, ProductInfo, ProductData
  - NameString, Name
  - Cusomer, CusomerInfo, CustomerObject
  - moneyAmount, money
  - theMessage, message

### 발음하기 쉬운 이름을 사용하라
```java
// Bad
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
}
```
```java
// Good
class Customer {
    private Date generatonTimestamp;
    private Date modificationTimestamp;
    private final String recordId = "102";
}
```

### 검색하기 쉬운 이름을 사용하라
- 짧은 이름보다는 긴 이름을 사용하자
- 이름 길이는 범위 크기(scope)에 비례해야 한다.
```java
// Bad
int s = 0;
for (int j = 0; j < 34; j++) {
    s += (t[j] * 4) / 5;
}
```
```java
// Good
int realDaysPerIdealDay = 4;
final int WORK_DAYS_PER_WEEK = 5;
final int NUMBER_OF_TASKS = 34;
int sum = 0;
for (int j = 0; j < NUMBER_OF_TASKS; j++) {
    int realTaskDays = taskEstimate[j] * realDaysPerIdealDay;
    int realTaskWeeks = (realTaskDays / WORK_DAYS_PER_WEEK);
    sum += realTaskWeeks;
}
```

### 인코딩을 피하라
- [헝가리식 표기법](https://en.wikipedia.org/wiki/Hungarian_notation)
  - 컴파일러와 IDE의 발전으로 인해 사용하지 마라
- 접두어(또는 접미어)를 사용하지 마라
  - `string m_dsc` 보다는 `string description`을 사용하라
- 인터페이스 클래스와 구현 클래스
  - 인터페이스와 구현클래스는 구분을 위해 인코딩이 필요하다
    - `IShapeFactory`, `ShapeFactory` 또는
    - `ShapeFactory`, `ShapeFactoryImp` -> 선택

### 자신의 기억력을 자랑하지 마라
- 코드 읽는이가 변수 이름을 머리속으로 재변환하게 하지마라
- 루프 범위가 아주 작고 다른 이름과 충돌하지 않을 때는 문자 하나의 변수 이름(`i`, `j`, `k`)는 괜찮다.
  - 그 외에는 코드 독자가 머리속으로 실제 개념으로 변환이 필요하게하므로 적절하지 못하다.

### 클래스 이름과 메서드 이름
- 클래스 이름, 객체 이름: 명사, 명사구
  - 좋은 예) Customer, WikiPage, Account, AddressParser
  - 나쁜 예) Manager, Processor, Data, Info와 같은 단어, 동사
- 메서드 이름: 동사, 동사구
    - 좋은 예) postPayment, deletePage, save
    - 접근자, 변경자, 조건자에는 `get`, `set`, `is`을 변수명 앞에 붙인다.
- 생성자를 오버로딩할 때는 정적 팩토리 메서드를 사용한다. 이때 메서드명은 인수를 설명하는 이름을 사용한다.
  ```java
  public class Cell {
      double value;
  
      private Cell(double value) {
          this.value = value;
      }
  
      public static Cell FromRealNumber(double x) {
          return new Cell(x);
      }
  }
  
  Cell cell = Cell.FromRealNumber(23.0);
  ```

### 기발한 이름은 피하라
- 기발하거나 재미난 이름보다는 명료한 이름을 선택하라
- 의도를 분명하고 솔직하게 표현하라

### 한 개념에 한 단어를 사용하라
- 메서드 이름은 독자적이고 일관적이어야 한다.
- 이름이 다르면 독자는 당연히 클래스가 다르고, 타입도 다르리라 생각한다.
  - 예를 들어, 동일 코드 기반에 `controller`, `manager`, `driver`를 섞어 쓰면 혼란스럽다.

### 말 장난을 하지 마라
- 다른 개념에 같은 단어를 사용하지 마라
  - 기존 값 2개를 더하거나 이어서 새로운 값을 만드는 메서드를 `add()`로 만들었다고 해서, 집합에 값 하나를 추가하는 메서드를 `add()`로 이름 짓는건 옳지 않다.
- 프로그래머는 집중적인 탐구가 필요한 코드가 아니라 대충 훑어봐도 이해할 코드 작성이 목표다.

### 해법 영역에서 가져온 이름을 사용하라
- 전산 용어, 알고리즘 이름, 패턴 이름, 수학 용어 등은 사용해도 괜찮다.
- 예를 들어, `JobQueue`라는 이름는 프로그래머라면 이해한다.
- 적절한 프로그래머 용어가 없다면 문제 영역(domain)에서 이름을 가져온다.

### 의미 있는 맥락을 추가하라
- 클래스, 함수, 이름 공간(namespace)에 넣어서 맥락(context)을 부여하라
- 모든 방법이 실패하면, 마지막 수단으로 접두어를 붙인다.
```java
// Bad
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluralModifier;
    if (count == 0) {  
        number = "no";  
        verb = "are";  
        pluralModifier = "s";  
    }  else if (count == 1) {
        number = "1";  
        verb = "is";  
        pluralModifier = "";  
    }  else {
        number = Integer.toString(count);  
        verb = "are";  
        pluralModifier = "s";  
    }
    String guessMessage = String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );

    print(guessMessage);
}
```
```java
// Good
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;

    public String make(char candidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );
    }

    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count);
        }
    }

    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }

    private void thereIsOneLetter() {
        number = "1";
        verb = "is";
        pluralModifier = "";
    }

    private void thereAreNoLetters() {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
```

### 불필요한 맥락을 없애라
- `Gas Station Deluxe`라는 애플리케이션을 만든다고 해서, 모든 클래스 이름 앞에 `GSD`를 붙이는 것은 좋지 않다.
- `accoundAddress`와 `customerAddress`는 `Address` 클래스의 인스턴스로는 좋은 이름이 아니다.