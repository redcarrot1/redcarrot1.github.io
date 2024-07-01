---
title: "LocalDateTime은 테스트하기 어렵다."
date: 2024-07-01 10:00:00 +09:00
categories: [Spring]
tags:
  [
    java, Spring, LocalDateTime, LocalDate, JpaAuditing
  ]
---

## Intro
백엔드 서버를 개발하면 시간 관련 로직을 자주 사용합니다. 게시물 작성 시각이나 특정 날짜로 필터링하는 기능은 주변 서비스에서 쉽게 찾아볼 수 있습니다.
하지만 시간 관련 로직은 테스트하기 어렵습니다. 특히 `LocalDateTime.now()` 처럼 static 메서드를 직접 사용한다면 mocking하기 복잡해집니다.<br>
또한 JpaAuditing 기술을 사용해서 CreatedDate, LastModifiedDate를 자주 사용하실 겁니다. 우리의 비즈니스 로직이 해당 필드에 의존한다면 어떤가요?

본 게시물에서는 시간 관련 로직의 테스트에 대해 고민해 보려고 합니다.


## LocalDateTime.now(), LocalDate.now()
지금부터 12시간 이내의 게시물만 조회하는 기능을 제공하려고 합니다.
```java
public List<Post> findPostsAfter12hoursAgo() {
    LocalDateTime now = LocalDateTime.now();
    LocalDateTime twelveHoursAgo = now.minusHours(12);
    return postRepository.findByCreatedAtGreaterThanEqual(twelveHoursAgo);
}
```

이제 해당 Serivce Layer부터 그 하위 플로우까지 통합 테스트를 진행하고 싶습니다.<br>
특히 **경계값**을 테스트하고 싶습니다. 정확히 12시간이 지난 게시물이 정상적으로 조회되는지 확인하려고 합니다.

```java
// 테스트가 실패한다.
@Test
@DisplayName("정확히 12시간 이전 게시글은 정상적으로 조회된다.")
void findPostsAfter12hoursAgo() {
    // given
    Post post = new Post("title", "content", LocalDateTime.now().minusHours(12));
    postRepository.save(post);

    // when
    List<Post> posts = postService.findPostsAfter12hoursAgo();

    // then
    assertThat(posts).hasSize(1)
            .extracting("id", "title", "content")
            .containsExactly(tuple(post.getId(), post.getTitle(), post.getContent()));
}
```
위 테스트는 실패합니다. 호출되는 service 메서드 내부와 given 절의 시점이 동일하지 않기 때문입니다.

### Ver 1
첫 번째로는 테스트하기 어려운 코드를 바깥으로 밀어내는 방법입니다.
```java
public List<Post> findPostsAfterAt(LocalDateTime dateTime) {
    return postRepository.findByCreatedAtGreaterThanEqual(dateTime);
}
```
상황에 따라선 아주 좋은 방안이 될 것 같습니다. 특히 도메인 객체처럼 값객체의 성질을 가진 클래스에서는 파라미터로 받는 게 좋을 것 같네요.<br>
근데 **어디까지 밀어내야 할까요?** controller까지 밀수도 있고, 아예 request부터 받는 방법도 있겠네요. 물론 이렇게 되면 필드 정의를 약간 바꿔야 될 수 있습니다. 예를 들어(위 예시 코드와 별개로), '게시물을 저장하는 로직이 실행되는 시점'보다는 '요청 시점'이 더 알맞겠네요.

controller까지 밀어낸다면, 해당 레이어를 포함한 통합테스트는 어떻게 해야 하나요? controller는 통합 테스트 대신, 단위(unit) 테스트만 진행하면 될 것 같네요. 더 좋은 방법이 없을까요?


### Ver 2
mockito 3.4.0 이상부터 static 클래스의 mocking을 지원합니다.

```java
@Test
@DisplayName("정확히 12시간 이전 게시글은 정상적으로 조회된다.")
void findPostsAfter12hoursAgo2() {
    // given
    LocalDateTime dateTime = LocalDateTime.now();
    Post post = new Post("title", "content", dateTime.minusHours(12));
    postRepository.save(post);

    List<Post> posts;
    try (MockedStatic<LocalDateTime> dateTimeMocked = Mockito.mockStatic(LocalDateTime.class)) {
        dateTimeMocked.when(LocalDateTime::now).thenReturn(dateTime);

        // when
        posts = postService.findPostsAfter12hoursAgo();
    }

    // then
    assertThat(posts).hasSize(1)
            .extracting("id", "title", "content")
            .containsExactly(tuple(post.getId(), post.getTitle(), post.getContent()));
}
```
다행히 테스트는 통과합니다. 하지만 static 메서드 목킹으로 인해 코드가 많이 추가됐습니다. static 메서드를 사용하지 않는 방향으로 만들어보겠습니다.

### Ver 3
먼저 DateTimeService 클래스를 추가하고, 빈으로 등록합니다.

```java
@Component
public class DateTimeService {
    
    public LocalDateTime getNow() {
        return LocalDateTime.now();
    }

}
```

우리의 비즈니스 로직에는 DateTimeService를 주입받아, 해당 클래스의 메서드를 사용합니다.
```java
public List<Post> findPostsAfter12hoursAgo() {
    LocalDateTime now = dateTimeService.getNow();
    LocalDateTime twelveHoursAgo = now.minusHours(12);
    return postRepository.findByCreatedAtGreaterThanEqual(twelveHoursAgo);
}
```

이제 테스트를 해보겠습니다.
```java
@MockBean
private DateTimeService dateTimeService;

@Test
@DisplayName("정확히 12시간 이전 게시글은 정상적으로 조회된다.")
void findPostsAfter12hoursAgo3() {
    // given
    LocalDateTime dateTime = LocalDateTime.now();
    Post post = new Post("title", "content", dateTime.minusHours(12));
    postRepository.save(post);

    when(dateTimeService.getNow()).thenReturn(dateTime);

    // when
    List<Post> posts = postService.findPostsAfter12hoursAgo();

    // then
    assertThat(posts).hasSize(1)
            .extracting("id", "title", "content")
            .containsExactly(tuple(post.getId(), post.getTitle(), post.getContent()));
}
```
예상대로 테스트는 통과합니다. 그리고 더 깔끔하고 읽기 쉬운 테스트 코드가 되었습니다.

> 그냥 static mocking하면 되지, 왜 굳이 복잡하게 클래스를 하나 더 만드나요?

static method의 mocking 자체를 안티패턴으로 보는 경우가 많습니다. (저는 근거에 대해서 크게 동의하진 않지만.)<br>
하지만 저에게 근거는 아래 2가지입니다. 특히 1번이 주된 이유입니다.
1. test 코드가 길어진다.
2. 자원 할당, 해제해야 한다. (차선책으로 try-with-resources 사용)


## JpaAuditing을 비즈니스 로직에서 의존하지 말자
일반적으로 엔티티를 저장하는 시점이나 수정 시점을 컬럼에 추가하여 함께 저장합니다. 많은 분이 아래 코드를 작성해 보셨을 겁니다.
```java
@Getter
@MappedSuperclass
@EntityListeners(value = {AuditingEntityListener.class})
public abstract class BaseTimeEntity {

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

JpaAuditing은 `LocalDateTime.now()`보다 더 독합니다. 도대체 목킹을 어떻게 해야 할까요.
사실 불가능한 건 아닙니다. 내부적으로 `DateTimeProvider`를 사용하기 때문에 이 클래스를 목킹하면 됩니다.
```java
@MockBean
private DateTimeProvider dateTimeProvider;

@SpyBean
private AuditingHandler auditingHandler;

@Test
@DisplayName("정확히 12시간 이전 게시글은 정상적으로 조회된다.")
void findPostsAfter12hoursAgo() {
    // given
    LocalDateTime dateTime = LocalDateTime.now();

    auditingHandler.setDateTimeProvider(dateTimeProvider);
    when(dateTimeProvider.getNow())
            .thenReturn(Optional.of(dateTime.minusHours(12)));

    Post post = new Post("title", "content");
    post2Repository.save(post);

    when(dateTimeService.getNow()).thenReturn(dateTime);

    // when
    List<Post> posts = postService.findPostsAfter12hoursAgo();

    // then
    assertThat(posts).hasSize(1)
            .extracting("id", "title", "content")
            .containsExactly(tuple(post.getId(), post.getTitle(), post.getContent()));
}
```
참고로 BaseTimeEntity에 값이 수정되는 시점은 `@PrePersist`, `@PreUpdate` 에 의해 결정됩니다. (`AuditingEntityListener` 내부에 해당 애노테이션들이 사용됩니다.) 관련 내용은 [하이버네이트 공식 문서](https://docs.jboss.org/hibernate/orm/6.5/userguide/html_single/Hibernate_User_Guide.html#events-jpa-callbacks)를 참고해주세요.



## Outro
테스트 코드는 문서이기도 합니다. 읽는 사람이 머릿속으로 로직을 분석하며 읽게 하는 것보다는, 심플하고 직관적으로 작성하는게 좋습니다.
이런 이유로 테스트 코드에서 given 절이 길어지고 중복 코드가 발생해도 복잡하게 분리하지 않는 경우도 많습니다.

애플리케이션이 복잡해질수록 테스트 코드는 중요해지는 것 같습니다. 특히 기능 수정이나 리팩토링에서 테스트 코드가 갖는 힘은 대단합니다. 처음에는 시간 낭비인 것 같지만, 결국에는 가장 빠른 길입니다.<br>
보통 사이드 프로젝트는 기능 수정할 일이 많지 않습니다. 일회성이 크기 때문입니다. 지속적으로 운영되는 서비스를 만들어보셨다면 테스트가 주는 힘을 경험하셨으리라 생각합니다. 고맙다 테스트야! 를 외치며 포스팅을 마무리합니다.