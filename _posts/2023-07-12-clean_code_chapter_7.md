---
title: Chapter 07 - 오류 처리
date: 2023-07-12 12:00:00 +09:00
categories: [Book, Clean Code]
tags:
  [
    clean code,
  ]
---

## 오류 코드보다 예외를 사용하라
- 오류 코드를 사용하면 호출자 코드가 복잡해지고, 잊어버리기 쉽다.
- 오류가 발생하면 예외를 던지는 편이 낫다. (호출자 코드가 더 깔끔해진다.)
  - 논리와 오류 처리 코드가 뒤섞이지 않기 때문이다.

```java
// Bad
public class DeviceController {
  ...
  public void sendShutDown() {
    DeviceHandle handle = getHandle(DEV1);
    // 디바이스 상태를 점검한다.
    if (handle != DeviceHandle.INVALID) {
      // 레코드 필드에 디바이스 상태를 저장한다.
      retrieveDeviceRecord(handle);
      // 디바이스가 일시정치 상태가 아니라면 종료한다.
      if (record.getStatus() != DEVICE_SUSPENDED) {
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
      } else {
        logger.log("Device suspended. Unable to shut down");
      }
    } else {
      logger.log("Invalid handle for: " + DEV1.toString());
    }
  }
  ...
}
```

```java
// Good
public class DeviceController {
  ...
  public void sendShutDown() {
    try {
      tryToShutDown();
    } catch (DeviceShutDownError e) {
      logger.log(e);
    }
  }
    
  private void tryToShutDown() throws DeviceShutDownError {
    DeviceHandle handle = getHandle(DEV1);
    DeviceRecord record = retrieveDeviceRecord(handle);
    pauseDevice(handle); 
    clearDeviceWorkQueue(handle); 
    closeDevice(handle);
  }
  
  private DeviceHandle getHandle(DeviceID id) {
    ...
    throw new DeviceShutDownError("Invalid handle for: " + id.toString());
    ...
  }
  ...
}
```

## Try-Catch-Finally 문부터 작성하라
- 어떤 면에서 try 블록은 트랜잭션과 비슷하다.
  - try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
  try {
    FileInputStream stream = new FileInputStream(sectionName);
    stream.close();
  } catch (FileNotFoundException e) {
    throw new StorageException("retrieval error", e);
  }
  return new ArrayList<RecordedGrip>();
}

// Test 성공
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
  sectionStore.retrieveSection("invalid - file");
}
```

## 미확인(unchecked) 예외를 사용하라
- checked 예외는 OCP(Open Closed Principle)를 위반한다.
  - 하위 메서드에서 던진 checked 예외를 상위 메소드들에서 catch로 정의해야하기 때문이다.
  - 또는 함수 선언부에 throw 절을 추가해야 한다.
  - 결과적으로 최하위 단계에서 최상위 단계까지 연쇄적인 수정이 일어난다.
- 또한 checked 예외는 하위 함수가 던지는 예외 타입을 알아야 하므로 캡슐화가 깨진다.

## 예외에 의미를 제공하라
- 예외를 던질 때는 전후 상황을 충분히 덧붙인다.
- 자바는 모든 예외에 호출 스택을 제공하지만, 의도를 파악하기에는 부족하다.
- 따라서 오류 메시지에 정보를 담아 예외와 함께 던진다.
  - 실패한 연산 이름, 실패 유형 등
- catch 블록에서 오류를 로깅하도록 충분히 정보를 념겨주자

## 호출자를 고려해 예외 클래스를 정의하라
- 아래와 같은 코드는 호출 함수가 던질 예외를 모두 잡아 낸다.
- catch 문의 내용이 거의 같은 것을 주목 (중복이 심하다.)

```java
// Bad

ACMEPort port = new ACMEPort(12);
try {
  port.open();
} catch (DeviceResponseException e) {
  reportPortError(e);
  logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
  reportPortError(e);
  logger.log("Unlock exception", e);
} catch (GMXError e) {
  reportPortError(e);
  logger.log("Device response exception");
} finally {
  ...
}
```

- 함수가 던지는 예외를 잡아 변환하는 wrapper 클래스를 사용해보자.
- 특히 외부 라이브러리를 사용하는 경우에 wrapper 클래스는 많은 이점을 가져온다.
  - 라이브러리 교체와 변경에 대응하기 쉽다.
  - 라이브러리를 목킹하여 테스트하기 쉽다.
  - 특정 업체가 설계한 방식에 발목 잡히지 않는다.

```java
// Good
  
LocalPort port = new LocalPort(12);
try {
  port.open();
} catch (PortDeviceFailure e) {
  reportError(e);
  logger.log(e.getMessage(), e);
} finally {
  ...
}

// LocalPort은 wrapper 클래스이다.
public class LocalPort {
  private ACMEPort innerPort;
  public LocalPort(int portNumber) {
    innerPort = new ACMEPort(portNumber);
  }
  
  public void open() {
    try {
      innerPort.open();
    } catch (DeviceResponseException e) {
      throw new PortDeviceFailure(e);
    } catch (ATM1212UnlockedException e) {
      throw new PortDeviceFailure(e);
    } catch (GMXError e) {
      throw new PortDeviceFailure(e);
    }
  }
  ...
}
```

## 정상 흐름을 정의하라
- 비즈니스 로직 중간에 예외 처리를 넣으면 정상 흐름을 파악하는데 어려울 수 있다.
- Special Case Pattern을 사용하자

```java
// Bad
// 로직을 이해하는데 어렵다.
try {
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
  m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
  m_total += getMealPerDiem();
}
```

```java
// Good
// 단순히 expenses.getTotal()만 더해주면 된다.
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();


public class PerDiemMealExpenses implements MealExpenses {
  public int getTotal() {
    // getMealPerDiem() 내용을 여기에 작성한다.
  }
}

public class ExpenseReportDAO {
  public MealExpenses getMeals(int employeeId) {
    MealExpenses expenses;
    try {
      expenses = expenseReportDAO.getMeals(employee.getID());
    } catch(MealExpensesNotFound e) {
      expenses = new PerDiemMealExpenses();
    }
    
    return expenses;
  }
  ...
}
```

## null을 반환, 전달하지 마라
- 메서드에서 null을 반환하지 말고, 예외를 던지거나 Special Case 객체를 반환하라
- 메서드 인수로 null을 전달하지 마라.
  - 정상적인 인수로 null을 기대하는 API가 아니라면, null을 전달하지 마라
