---
title: Chapter 13 - 동시성
date: 2023-07-19 12:00:00 +09:00
categories: [Book, Clean Code]
tags:
  [
    clean code,
  ]
img_path: /assets/img/clean_code/
---
## 동시성이 필요한 이유
- 동시성은 무엇(what)과 언제(when)을 분리하는, 결합(coupling)을 없애는 전략이다.
- 프로그램의 구조와 효율 개선을 위해 동시성을 이용한다.
- 동시성은 어렵다. 각별히 주의하지 않으면 난감한 상황에 처한다.

**미신과 오해**
- 동시성은 항상 성능을 높여준다.
  - 대기 시간이 아주 길어 여러 스레드가 프로세서를 공유할 수 있거나, 여러 프로세스가 독립적으로 계산이 충분히 많은 경우에만 성능이 높아진다.
- 동시성을 구현해도 설계는 변하지 않는다.
  - 단일 스레드 시스템과 다중 스레드 시스템은 설계가 판이하게 다르다.
- 웹 또는 EJB 컨테이너를 사용하면 동시성을 이해할 필요가 없다.
  - 컨테이너가 어떻게 동작하는지, 동시 수정과 데드락 등과 같은 문제를 피할 수 있는지 알아야만 한다.

**타당한 생각**
- 동시성은 다소 부하를 유발한다.
  - 성능 측면에서 부하가 걸리며, 코드도 더 짜야 한다.
- 동시성은 복잡하다.
- 일반적으로 동시성 버그는 재현하기 어렵다.
- 동시성을 구현하려면 흔히 근본적인 설계 전략을 재고해야 한다.

## 난관
```java
public class X{
  private int lastIdUsed;

  public int getNextId(){
    return ++lastIdUsed;
  }
}
```
- 하나의 인스턴스를 두 스레드가 동시에 `getNextId()`를 호출한다고 가정하자.
  - 같은 변수에 동시에 참조하면 `race condition`이 발생할 수 있다.(잘못된 결과)

## 동시성 방어 원칙
- 단일 책임 원칙(SRP)
  - 동시성 관련 코드는 다른 코드와 분리해야 한다.
- 따름 정리 : 자료 범위를 제한하라
  - 임게 영역(critical section)을 `synchronized` 키워드로 보호하라
  - 자료를 캡슐화하여, 공유 자료를 최대한 줄여라
- 따름 정리 : 자료 사본을 사용하라
  - 공유 자료를 줄이려면 처음부터 공유하지 않는 방법이 제일 좋다.
  - 객체를 복사해 읽기 전용으로 사용하는 방법이 있다.
  - 복사 비용이 들더라도, 동기화(또는 내부 잠금)을 없애 절약한 수행 시간이 더 클 가능성이 높다.
- 따름 정리 : 스레드는 가능한 독립적으로 구현하라
  - 각 스레드는 다른 스레드와 자료를 공유하지 않도록 하라

## 라이브러리를 이해하라
- 자바 5는 동시성 라이브러리가 많이 추가됐다.
  - `ReentrantLock` : 한 메서드에서 잠그고, 다른 메서드에서 푸는 락(lock)이다.
  - `Semaphore` : 전형적인 세마포다. 개수가 있는 락이다.
  - `CountDownLatch` : 지정한 수만큼 이벤트가 발생하고 나서야 대기 중인 스레드를 모두 해제하는 락이다.

## 실행 모델을 이해하라
- 기본적인 용어

| Name                         | Description  |
| :--------------------------- | :----------- |
| Bound Resources              | 다중 스레드 환경에서 사용하는 제한적인 자원. 예로 데이터베이스 연결, 길이가 일정한 읽기/쓰기 버퍼가 있다. |
| Mutual Exclusion             | 한 시점에 공유 자원에 접근할 수 있는 스레드는 단 하나이다. |
| Starvation                   | 한 스레드나 여러 스레드가 굉장히 오랫동안 혹인 영원히 자원을 기다린다. |
| Deadlock                     | 여러 스레드가 서로가 끝나기를 기다린다. 모든 스레드가 각기 필요한 자원을 다른 스레드가 점유하는 바람에 어느쪽도 더 이상 진행하지 못한다. |
| Livelock | 락을 거는 단계에서 각 쓰레드가 서로를 방해한다. 스레드는 계속해서 진행하려 하지만, 공명으로 인해, 굉장히 오랫동안 혹은 영원히 진행하지 못한다. |

- 실행 모델
  - [생산자-소비자](https://ko.wikipedia.org/wiki/%EC%83%9D%EC%82%B0%EC%9E%90-%EC%86%8C%EB%B9%84%EC%9E%90_%EB%AC%B8%EC%A0%9C)
  - [읽기-쓰기](https://ko.wikipedia.org/wiki/%EB%8F%85%EC%9E%90-%EC%A0%80%EC%9E%90_%EB%AC%B8%EC%A0%9C)
  - [식사하는 철학자들](https://ko.wikipedia.org/wiki/%EC%8B%9D%EC%82%AC%ED%95%98%EB%8A%94_%EC%B2%A0%ED%95%99%EC%9E%90%EB%93%A4_%EB%AC%B8%EC%A0%9C)

## 동기화하는 메서드 사이에 존재하는 의존성을 이해하라
- 동기화하는 메서드 사이에 의존성이 존재하면 동시성 코드에 찾아내기 어려운 버그가 생긴다.
- 공유 클래스 하나에 동기화된 메서드가 여럿이라면 구현이 올바른지 확인하기 어렵다.

**권고사항**: *공유 객체 하나에는 메서드 하나만 사용하라.*

만일 공유 객체 하나에 여러 메서드가 필요한 상황이라면 아래 세 가지 방법을 고현한다.

**클라이언트 기반 잠금(Client-Based Locking)**: 클라이언트에서 첫 번째 메서드를 호출하기 전에 서버를 잠근다. 마지막 메서드를 호출할 때까지 잠금을 유지한다.

**서버 기반 잠금(Server-Based Locking)**: 서버를 잠그고 모든 메서드를 호출한 후 잠금을 해제하는 메서드를 구현한다. 클라이언트는 이 메서드를 호출한다.

**중계된 서버(Adapted Server)**: 잠금을 수행하는 중간 단계를 생성한다. (원래 서버는 변경하지 않는다.)

## 동기화하는 부분을 작게 만들어라
- 임계영역(critical section)은 반드시 보호해야 하지만, 여기저기 Synchronized 문을 남발하면 안된다.
- 코드를 짤 때는 임계영역의 크기를 작게, 수를 최대한 줄여야 한다.


## 올바른 종료 코드는 구현하기 어렵다
- 잠시 돌다 깔끔하게 종료하는 시스템은 올바로 구현하기 어렵다.
- 가장 흔히 발생하는 문제가 데드락이다.(스레드가 종료되지 않는다.)
- 이는 생각보다 어려우므로 이미 나온 알고리즘을 검토하라

## 스레드 코드 테스트하기 
- 테스트는 정확성을 보장하지 않지만, 충분한 테스트는 위험을 낮춘다.
- 문제를 노출하는 테스트 케이스를 작성하고, 프로그램 설정과 부하를 바꿔가며 자주 돌려라
- 스레드 코드에 있는 버그를 재현하기 아주 어렵다.(테스트 실패를 일회성 문제로 무시하지 마라)
