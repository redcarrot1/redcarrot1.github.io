---
title: Chapter 09 - 단위 테스트
date: 2023-07-13 12:00:00 +09:00
categories: [Book, Clean Code]
tags:
  [
    clean code,
  ]
---

## TDD 법칙 세 가지
1. 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
3. 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

- 개발과 테스트가 대략 30초 주기로 묶인다.
- 매일 수십 개, 매달 수백 개, 매년 수천 개에 달하는 테스트 케이스가 나오고, 실제 코드를 전부 테스트하게 된다.
- 하지만 방대한 테스트 코드는 심각한 관리 문제를 유발하기도 한다.

## 깨끗한 테스트 코드 유지하기
- 실제 코드가 진화하면 테스트 코드도 변해야 한다.
- 테스트 코드가 지저분할수록 변경하기 어려워진다.
- 테스트 코드는 실제 코드 못지 않게 깨끗하게 짜야 한다.

### 테스트는 유연성, 유지보수성, 재사용성을 제공한다.
- 아키텍터가 유연하고, 설계를 아무리 잘 나눴더라도 개발자는 테스트 케이스가 없으면 버그가 숨어들까 변경을 주저한다.
- 테스트 케이스가 있으면 변경이 쉬워진다. 안심하고 아키텍처와 설계를 개선할 수 있다.

## 깨끗한 테스트 코드
- 깨끗한 테스트 코드를 만들기 위해서는 가독성이 중요하다.
- 가독성을 위해서는 명료성, 단순성, 풍부한 표현력이 필요하다. 테스트는 최소의 표현으로 많은 것을 나타내야 한다.
- BUILD-OPERATE-CHECK 패턴 : 테스트를 명확히 세 부분으로 나눈다.
  - 첫 부분 : 테스트 자료를 만든다.
  - 두 번째 부분 : 테스트 자료를 조작한다.
  - 세 번째 부분 : 조작한 결과가 올바른지 확인한다.

### 이중 표준
- 테스트 API 코드에 적용하는 표준은 실제 코드에 적용하는 표준과 학실히 다르다.
- 단순하고, 간결하고, 표현력이 풍부해야 하지만, 실제 코드만큼 효율적일 필요는 없다.
- 실제 환경에서는 절대로 안 되지만 테스트 환경에서는 전혀 문제없는 방식이 있다.
  - 예를 들어, StringBuffer을 사용하지 않고 string을 연속해서 덧붙이는 경우
  - 대개 메모리나 CPU 효율과 관련 있는(실시간 임베디드 등) 경우다.
  - 이것은 코드의 깨끗함과는 **철저히** 무관하다.

```java
// Bad
// 오른쪽 함수를 읽고, 왼쪽 assertTrue를 읽어야 한다. 가독성이 낮다.
@Test
public void turnOnLoTempAlarmAtThreashold() throws Exception {
  hw.setTemp(WAY_TOO_COLD); 
  controller.tic(); 
  assertTrue(hw.heaterState());   
  assertTrue(hw.blowerState()); 
  assertFalse(hw.coolerState()); 
  assertFalse(hw.hiTempAlarm());       
  assertTrue(hw.loTempAlarm());
}
```

```java
// Good
// "HBchL"은 대문자는 켜짐, 소문자는 꺼짐을 의미하고, 그 순서는 heater, blower, .. 순서이다.
// 실제 코드에서는 규칙 위반이지만, 테스트 코드에서는 적절하다.
@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
  wayTooCold();
  assertEquals("HBchL", hw.getState()); 
}

public String getState() {
  String state = "";
  state += heater ? "H" : "h"; 
  state += blower ? "B" : "b"; 
  state += cooler ? "C" : "c"; 
  state += hiTempAlarm ? "H" : "h"; 
  state += loTempAlarm ? "L" : "l"; 
  return state;
}
```

## 테스트 당 assert 하나
- 테스트를 쪼개 각자가 assert를 수행하면 읽기가 쉬워진다.
- 하지만 테스트를 분리하면 중복되는 코드가 많아진다.
- TEMPLATE METHOD 패턴을 사용하면 중복을 제거할 수 있다.
  - given/when 부분을 부모 클래스에 두고, then 부분을 자식 클래스에 두면 된다.
  - 또는 @Before 함수에 given/when 부분을 넣고, @Test 함수에 then 부분을 넣는다.
- 하지만 배보다 배꼽이 더 크다.(테스트가 너무 크다.)
- 여러 요소를 고려해볼 때, 함수 하나에 여러 assert 문을 사용하는 편이 좋다고 생각한다.(저자)
- 결론
  - 개념 당 assert 문 수를 최소로 줄여라
  - 테스트 함수 하나는 개념 하나만 테스트하라

```java
// given, when, then 관례
// 테스트를 분리하면 중복되는 코드가 많아진다.
public void testGetPageHierarchyAsXml() throws Exception { 
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
  
  whenRequestIsIssued("root", "type:pages");
  
  thenResponseShouldBeXML(); 
}

public void testGetPageHierarchyHasRightTags() throws Exception { 
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
  
  whenRequestIsIssued("root", "type:pages");
  
  thenResponseShouldContain(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
  ); 
}
```

## F.I.R.S.T
**빠르게(Fast)**<br>
테스트는 빨라야 한다. 테스트가 느리면 자주 돌릴 엄두를 못 낸다. 자주 돌리지 않으면 초반에 문제를 찾아내 고치지 못한다. 코드를 마음껏 정리하지도 못한다. 결국 코드 품질이 망가지기 시작한다.

**독립적으로(Independent)**<br>
각 테스트를 서로 의존하면 안 된다. 한 테스트가 다음 테스트가 실행될 환경을 준비해서는 안 된다. 각 테스트는 독립적으로 그리고 어떤 순서로 실행해도 괜찮아야 한다. 테스트가 서로에게 의존하면 하나가 실패할 때 나머지도 잇달아 실패하므로 원인을 진단하기 어려워지며 후반 테스트가 찾아내야 할 결함이 숨겨진다.

**반복가능하게(Repeatable)**<br>
테스트는 어떤 환경에서도 반복 가능해야 한다. 실제 환경, QA 환경, 버스를 타고 집으로 가는 길에 사용하는 노트북 환경(네트워크가 연결되지 않은)에서도 실행할 수 있어야 한다.

**자가검증하는(Self-Validating)**<br>
테스트는 성공 아니면 실패로 결과를 내야한다. 로그 파일을 읽게 만들어서는 안 된다. 통과 여부를 보려고 텍스트 파일 두 개를 수작업으로 비교하게 만들어서도 안 된다. 테스트가 스스로 성공과 실패를 가늠하지 않는다면 판단은 주관적이 되며 지루한 수작업 평가가 필요하게 된다.

**적시에(Timely)**<br>
테스트는 적시에 작성해야 한다. 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다. 실제 코드를 구현한 다음에 테스트 코드를 만들면 실제 코드가 테스트하기 어렵다는 사실을 발견할지도 모른다. 어떤 실제 코드는 테스트하기 너무 어렵다고 판명날지 모른다. 테스트가 불가능하도록 실제 코드를 설계할지도 모른다.