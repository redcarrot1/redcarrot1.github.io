---
title: Spring Transaction 1
date: 2024-01-11 10:00:00 +09:00
categories: [Spring-boot]
tags:
  [
    spring, Transaction
  ]
img_path: /assets/img/spring_transaction/
---

## 스프링 트랜잭션 추상화
데이터 접근 기술들은 트랜잭션을 처리하는 코드 자체가 다르다. 예를 들면, JDBC와 JPA는 트랜잭션을 사용하는 코드가 다르다.<br>
데이터 접근 기술을 바뀐다면 트랜잭션 관련 코드를 개발자가 모두 수정해야 한다. 스프링은 이런 문제를 해결하기 위해 `PlatformTransactionManager` 인터페이스를 만들어 트랜잭션 코드를 추상화했다.<br>
해당 인터페이스의 구현체는 스프링 부트가 데이터 접근 기술을 자동으로 인식해서 적절한 구현체를 빈으로 등록시켜준다. 따라서 개발자는 `PlatformTransactionManager` 인터페이스만 잘 알고있으면 된다.

![](1.png)

## 스프링 트랜잭션 사용 방식
### 선언적 트랜잭션 관리(Declarative Transaction Management)
- `@Transactional` 애노테이션을 이용해 AOP를 적용시킨다.
- 대부분 이 방식을 사용 (아래 모든 내용은 선언적 트랜잭션을 전제로 설명한다.)
- 스프링이 자동으로 비즈니스 로직 앞, 뒤에 트랜잭션 관련 코드를 넣은 프록시 클래스를 빈으로 등록한다
- AOP 관련해서는 이후에 다른 게시물로 알아볼 예정이다.

### 프로그래밍 방식의 트랜잭션 관리(Programmatic Transaction Management)
- 트랜잭션 매니저나 트랜잭션 템플릿 등을 사용해서 직접 관련 코드를 작성하는 방식
- 애플리케이션 코드(비즈니스 로직)이 트랜잭션 코드와 강하게 결합되기 때문에 주로 사용되지 않는다.

```java
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition()); // 트랜잭션 시작
try {
    //비즈니스 로직
    bizLogic(fromId, toId, money);
    transactionManager.commit(status); //성공시 커밋
} catch (Exception e) {
    transactionManager.rollback(status); //실패시 롤백
    throw new IllegalStateException(e);
}
```

## 트랜잭션 적용 우선순위
스프링은 항상 더 구체적이고 자세한 것이 높은 우선순위를 가진다. 트랜잭션뿐만 아니라 대부분의 우선순위 처리를 동일한 원칙으로 다룬다.<br>
트랜잭션의 경우 아래와 같은 우선순위를 가진다.
1. 클래스의 메서드 (우선순위가 가장 높다.)
2. 클래스의 타입
3. 인터페이스의 메서드
4. 인터페이스의 타입 (우선순위가 가장 낮다.)
readOnly와 같은 옵션값은 항상 높은 우선순위의 값을 따르게 된다. 또한 인터페이스에도 `@Transactional`을 사용할 순 있지만 권장하는 방식은 아니라고 한다.

<br>

## 트랜잭션 AOP 주의 사항 - 프록시 내부 호출(중요)
`@Transactional`를 사용하는 트랜잭션 AOP는 프록시를 사용한다. 여기서 생기는 한계점이 있는데, 바로 메서드 내부 호출은 프록시 코드가 적용되지 않는다는 점이다. 아래 코드를 보자

```java
@Slf4j
@SpringBootTest
public class InternalCallV1Test {
    @Autowired
    CallService callService;

    @Test
    void externalCall() {
        callService.external();
    }

    @TestConfiguration
    static class InternalCallV1Config {
        @Bean
        CallService callService() {
            return new CallService();
        }
    }

    @Slf4j
    static class CallService {
        public void external() {
            log.info("call external");
            printTxInfo();
            internal();
        }

        @Transactional
        public void internal() {
            log.info("call internal");
            printTxInfo();
        }

        private void printTxInfo() {
            boolean txActive =
                    TransactionSynchronizationManager.isActualTransactionActive();
            log.info("tx active={}", txActive);
        }
    }
}
/*
CallService   : call external
CallService   : tx active=false
CallService   : call internal
CallService   : tx active=false
*/
```

`internal()` 메서드에 트랜잭션을 걸었음에도 실제로는 걸리지 않았다. `external()`에서 내부 호출을 사용하고 있기 때문이다. 아래 그림을 참고하자.

![](2.png)

이런 문제를 해결하기 위한 가장 간단한 방법은 `internal()` 메서드를 별도의 클래스로 분리하는 것이다.

![](3.png)

참고로 스프링 AOP는 `public` 메서드에만 트랜잭션이 적용된다. 스프링 부트 3.0부터는 `protected`, `default` 메서드에서도 트랜잭션이 적용된다. 골자는 `private` 메서드에는 AOP가 적용되지 않는다는 것이다.(트랜잭션 적용이 무시된다.)

두번째 참고로는, `@PostContruct`가 붙은 메서드가 트랜잭션 AOP보다 빨리 호출된다. 따라서 해당 메서드에는 트랜잭션을 적용할 수 없다.<br>
해결 방법으로는 `@PostContruct` 대신 `@EventListener(value = ApplicationReadyEvent.class)`을 사용하면 된다. 해당 애노테이션은 모든 스프링 컨테이너가 완전히 생성된 이후에 호출되게 한다.

세번째 참고로는, `readOnly` 옵션이다. 데이터 읽기만 존재하는 경우 `readOnly=true`옵션을 통해 성능 최적화가 발생한다는 점은 알고있는 사실이다.<br>
궁금한 점은 `@Transactional`을 아예 붙이지 않는 것과의 성능비교이다. 영한님 피셜로는 readOnly가 DB 내부 최적화를 수행해서 성능상 이점이 있을수도 있고, 오히려 네트워크 통신이 1번 더 일어나서 성능상 불리한 경우도 있다고 한다. 따라서 DB마다 다르므로 성능 테스트 후에 결정해야 한다고 한다.

## 예외와 트랜잭션 커밋, 롤백
스프링 트랜잭션 AOP는 예외 종류에 따라 트랜잭션을 커밋하거나 롤백한다.
1. 언체크 예외(RuntimeException, Error) : 트랜잭션 롤백
2. 체크 예외(Exception) : 트랜잭션 커밋
3. 정상 응답 : 트랜잭션 커밋

스프링은 기본적으로 체크 예외는 비즈니스 의미가 있을 때 사용하고, 언체크 예외는 복구 불가능한 예외로 가정한다.<br>
이때 애노테이션 옵션으로 `rollbackFor`을 사용하면 체크 예외더라도 트랜잭션 롤백을 할 수 있다.

```java
// 런타임 예외 발생: 롤백
@Transactional
public void runtimeException() {
    throw new RuntimeException();
}

//체크 예외 발생: 커밋
@Transactional
public void checkedException() throws Exception {
    throw new Exception();
}


//체크 예외 rollbackFor 지정: 롤백
@Transactional(rollbackFor = Exception.class)
public void rollbackFor() throws Exception {
    throw new Exception();
}
```

## Reference
[김영한 스프링 DB 2편 - 데이터 접근 활용 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-2)