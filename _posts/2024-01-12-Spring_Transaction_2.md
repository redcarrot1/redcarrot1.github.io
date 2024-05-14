---
title: Spring Transaction 2 - propagation
date: 2024-01-12 10:00:00 +09:00
categories: [Spring-boot]
tags:
  [
    spring, Transaction
  ]
img_path: /assets/img/spring_transaction/
---

이번 포스팅에서는 스프링 트랜잭션 전파(propagation)에 대해 알아보자

## 기본 코드
먼저 본 포스팅에 기본이 되는 코드를 알아보자.

```java
@Slf4j
@SpringBootTest
public class BasicTxTest {
    @Autowired
    PlatformTransactionManager txManager;

    @TestConfiguration
    static class Config {
        @Bean
        public PlatformTransactionManager transactionManager(DataSource dataSource) {
            return new DataSourceTransactionManager(dataSource);
        }
    }

    @Test
    void code() {
        log.info("트랜잭션 시작");
        TransactionStatus status = txManager.getTransaction(new DefaultTransactionAttribute());
        log.info("트랜잭션 커밋 시작");
        txManager.commit(status);
        // txManager.rollback(status);
        log.info("트랜잭션 커밋 완료");
    }
}
```

- `PlatformTransactionManager` : [이전 포스팅](https://redcarrot1.github.io/posts/Spring_Transaction_1/)에 작성한 것처럼 스프링이 트랜잭션을 추상화한 인터페이스이다. 구현체는 여러가지 존재하는데, Spring-boot가 적절한 구현체를 빈으로 등록시켜준다. 위 코드에서는 JDBC 트랜잭션 구현체인 `DataSourceTransactionManager`을 사용했다.
- `DataSource` : DB 연결 정보를 저장하고 Connection을 등록 및 관리하는 인터페이스이다. 스프링 2.0부터 Hikari를 기본 구현체로 사용한다. Hikari는 Connection Pool 방식을 사용하여 효율적인 커넥션 사용을 도와준다.
- `txManager.getTransaction(new DefaultTransactionAttribute())` : 트랜잭션 매니저를 통해 트랜잭션을 시작(획득)한다.
- `txManager.commit(status)` : 트랜잭션을 커밋한다.
- `txManager.rollback(status)` : 트랜잭션을 롤백한다.
참고로 트랜잭션 동기화 매니저는 스프링 빈으로 등록되지는 않지만 모든 필드가 static으로 구성되어 있어 싱글톤처럼 사용된다.

<br>

## Case1 : 하나의 메서드에서 트랜잭션을 2번 수행
```java
@Test
void double_commit_rollback() {
    log.info("트랜잭션1 시작");
    TransactionStatus tx1 = txManager.getTransaction(new DefaultTransactionAttribute());
    log.info("트랜잭션1 커밋");
    txManager.commit(tx1);

    log.info("트랜잭션2 시작");
    TransactionStatus tx2 = txManager.getTransaction(new DefaultTransactionAttribute());
    log.info("트랜잭션2 롤백");
    txManager.rollback(tx2);
}
```
사실상 트랜잭션1과 트랜잭션2는 완전 별도의 트랜잭션이다. 트랜잭션1은 성공적으로 커밋하고 커넥션을 반환한다. 이후 트랜잭션2가 다시 커넥션을 얻고 성공적으로 롤백을 수행한다.

## Case2 : 중첩된 메서드 - 모두 커밋
트랜잭션 전파 기본옵션인 `REQUIRED`를 기준으로 설명한다.<br>

결론부터 이야기하자면 아래와 같은 원칙을 갖는다.
1. 하나라도 (논리) 트랜잭션이 롤백되면, 물리 트랜잭션이 롤백된다.
2. 모든 (논리) 트랜잭션이 커밋되면, 물리 트랜잭션이 커밋된다.

따라서 중첩된 메서드에 여러 트랜잭션이 걸려있는 경우, **모든 트랜잭션이 커밋되어야 전체적으로 커밋**된다고 생각하면 된다.

```java
@Test
void inner_commit() {
    log.info("외부 트랜잭션 시작");
    // Creating new transaction
    TransactionStatus outer = txManager.getTransaction(new DefaultTransactionAttribute());

    log.info("내부 트랜잭션 시작");
    // Participating in existing transaction
    TransactionStatus inner = txManager.getTransaction(new DefaultTransactionAttribute());
    log.info("내부 트랜잭션 커밋");
    txManager.commit(inner); // 실제로는 커밋 안함

    log.info("외부 트랜잭션 커밋");
    txManager.commit(outer); // 실제 커밋 수행
}
```

1. 외부 트랜잭션이 시작 이후 내부 트랜잭션이 시작된다. 내부 트랜잭션이 시작할 때 `기존 트랜잭션에 참여한다.`
2. 처음 트랜잭션을 시작한 외부 트랜잭션이 실제 물리 트랜잭션을 관리한다.
    - 실제로 내부 트랜잭션에서 커밋이 발생하더라도 커밋을 수행하지 않는다.

![](4.png)
![](5.png)


## Case3 : 중첩된 메서드 - 외부 롤백
중첩된 트랜잭션 중에 하나라도 롤백되면 전체를 롤백된다.
```java
@Test
void outer_rollback() {
    log.info("외부 트랜잭션 시작");
    TransactionStatus outer = txManager.getTransaction(new DefaultTransactionAttribute());

    log.info("내부 트랜잭션 시작");
    TransactionStatus inner = txManager.getTransaction(new DefaultTransactionAttribute());
    log.info("내부 트랜잭션 커밋");
    txManager.commit(inner);

    log.info("외부 트랜잭션 롤백");
    txManager.rollback(outer);
}
```

## Case4 : 중첩된 메서드 - 내부 롤백
중첩된 트랜잭션 중에 하나라도 롤백되면 전체를 롤백된다.
```java
@Test
void inner_rollback() {
    log.info("외부 트랜잭션 시작");
    TransactionStatus outer = txManager.getTransaction(new DefaultTransactionAttribute());

    log.info("내부 트랜잭션 시작");
    TransactionStatus inner = txManager.getTransaction(new DefaultTransactionAttribute());
    log.info("내부 트랜잭션 롤백");
    txManager.rollback(inner); // marking existing transaction as rollback-only

    log.info("외부 트랜잭션 커밋");
    assertThatThrownBy(() -> txManager.commit(outer))
            .isInstanceOf(UnexpectedRollbackException.class);
}
```

![](6.png)

내부 트랜잭션을 롤백하면 그 시점에 실제 물리 트랜잭션은 롤백하지 않는다. 대신에 기존 트랜잭션을 롤백 전용으로 표시한다.<br>
외부 트랜잭션이 커밋을 호출했지만, 전체 트랜잭션이 롤백 전용으로 표시되어 있기 때문에 최종적으로는 롤백을 수행한다.<br>
심지어 스프링은 `UnexpectedRollbackException` 런타임 에러까지 던져준다. 커밋을 시도했지만 기대하지 않은 롤백이 발생했다는 것을 명확하게 알려준다.


## 트랜잭션 완전히 분리하기 - REQUIRES_NEW
외부 트랜잭션과 내부 트랜잭션을 완전 분리하는 방법이 있다. 이 방법을 사용하면 어느 한쪽에서 롤백이 발생하더라도 정상 커밋은 그대로 진행한다.
```java
@Test
void inner_rollback_requires_new() {
    log.info("외부 트랜잭션 시작");
    TransactionStatus outer = txManager.getTransaction(new DefaultTransactionAttribute());
    log.info("outer.isNewTransaction()={}", outer.isNewTransaction()); // True

    log.info("내부 트랜잭션 시작");
    DefaultTransactionAttribute definition = new DefaultTransactionAttribute();
    definition.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
    TransactionStatus inner = txManager.getTransaction(definition);
    log.info("inner.isNewTransaction()={}", inner.isNewTransaction()); // True
    log.info("내부 트랜잭션 롤백");
    txManager.rollback(inner); // 롤백

    log.info("외부 트랜잭션 커밋");
    txManager.commit(outer); // 커밋
}
```

`TransactionDefinition.PROPAGATION_REQUIRES_NEW`을 사용하면 기존에 시작된 트랜잭션에 참여하지 않고, 완전 새로운 신규 트랜잭션으로 시작하게 된다.(물리 커넥션도 완전 별도로 사용한다.)<br>
커넥션을 여러개 사용하기 때문에 성능상 이슈가 생길 수 있다.

## 다양한 전파 옵션
실무에서는 대부분 `REQUIRED` 옵션을 사용한다. 그리고 아주 가끔 `REQUIRES_NEW` 을 사용하고, 나머지는 거 의 사용하지 않는다고 한다.

- REQUIRED
    - 기존 트랜잭션이 없으면 생성하고, 있으면 참여한다.
- REQUIRES_NEW
    - 항상 새로운 트랜잭션을 생성한다.
- SUPPORT
    - 기존 트랜잭션이 없으면, 없는대로 진행하고, 있으면 참여한다.
- NOT_SUPPORT
    - 트랜잭션을 지원하지 않는다. 기존 트랜잭션이 있다면 해당 트랜잭션은 보류한다.
- MANDATORY
    - 기존 트랜잭션이 없으면 예외가 발생한다. 기존 트랜잭션이 있다면 참여한다.
- NEVER
    - 트랜잭션을 사용하지 않는다. 기존 트랜잭션이 있다면 예외가 발생한다.
- NESTED
    - 기존 트랜잭션이 없으면 참여하고, 기존 트랜잭션이 있으면 중첩 트랜잭션을 만든다.
    - 중첩 트랜잭션은 외부 트랜잭션의 영향을 받지만, 중첩 트랜잭션은 외부에 영향을 주지 않는다.
    - JPA에서는 사용할 수 없다.

참고로 `isolation` , `timeout` , `readOnly` 는 트랜잭션이 처음 시작될 때만 적용된다. 트랜잭션에 참여하는 경우에는 적용되지 않는다

## Reference
[김영한 스프링 DB 2편 - 데이터 접근 활용 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-2)