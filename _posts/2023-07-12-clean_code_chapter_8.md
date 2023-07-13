---
title: Chapter 08 - 경계
date: 2023-07-12 12:00:00 +09:00
categories: [Book, Clean Code]
tags:
  [
    clean code,
  ]
---

외부 코드(오픈 소스 등)를 우리 코드에 깔끔하게 통합해야 한다.

## 외부 코드 사용하기
- 패키지 제공자나 프레임워크 제공자는 적용성을 최대한 넓히려 애쓴다.(그래야 많은 고객이 구매하니깐)
- 하지만 사용자는 자신의 요구에 집중하는 인터페이스를 원한다.

- 예시 : java.util.Map
  - 문제점1 : 해당 객체를 받으면 자유롭게 clear, add 등을 할 수 있다.(예상치 못한 결과)
  - 문제점2 : 저장 유형을 특정 객체로 제한하지 않는다. (not using generic)

  ```java
  Map sensors = new HashMap();
  Sensor s = (Sensor)sensors.get(sensorId);

  // generic을 사용하면 코드 가독성이 크게 높아진다.
  Map<String, Sensor> sensors = new HashMap<Sensor>();
  Sensor s = sensors.get(sensorId);

  // 진짜 Best는 클래스로 따로 빼서 Map을 숨기는 것
  public class Sensors {
    private Map sensors = new HashMap();
    public Sensor getById(String id) {
      return (Sensor)sensors.get(id);
    }
  }
  ```
  - 문제점3 : Map 인터페이스가 변할 경우, 수정할 코드가 상당히 많아진다.
    - 실제로 자바 5때 generic를 지원하면서 인터페이스가 변경됐다.

## 경계 살피고 익히기
- 외부 코드는 익히기 어렵고, 통합하기도 어렵다.
- 학습 테스트 : 먼저 간단한 테스트 케이스를 작성해 외부 코드 익히기
