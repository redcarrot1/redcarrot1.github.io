---
title: Presentation과 business layer의 의존성
date: 2024-03-12 10:00:00 +09:00
categories: [Spring-boot]
tags:
  [
    layered-architecture, dependency
  ]
img_path: /assets/img/etc/presentation_dependency/
---

 일반적으로 간단한 웹 서버 애플리케이션을 제작할 때 가장 먼저 떠오르는 아키텍처는 레이어드 아키텍처(layered-architecture)입니다.
단순하고 대중적이면서 비용도 적게 들기 때문에 초기 구축할 때 출발점으로 많이 선택합니다.<br>
 이번 포스팅에서는 presentation layer와 business layer의 의존성을 다룹니다. 모든 예시는 Spring boot를 기준으로 설명됩니다.

## Layered architecture
 먼저 레이어드 아키텍처를 간단히 살펴보겠습니다. 내부 컴포넌트는 논리적으로 수평한 레이어들로 구성됩니다.
각 레이어는 애플리케이션에서 프레젠테이션 로직, 비즈니스 로직 등의 주어진 역할을 수행하죠.
<br>
일반적으로 프레젠테이션, 비즈니스, 퍼시스턴스, 데이터베이스의 4개 표준 레이어로 구성합니다.
물론 규모에 따라 병합하기도 하며, 그 이상의 레이어로 구성하기도 합니다.(N-tier Architecture)

![](1.png){: width="500" height="" }
_https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch01.html_

### 관심사의 분리
 Layered Architecture의 중요한 특징은 **관심사의 분리(Separation of Concern)**입니다.
예를 들어 비즈니스 레이어는 데이터를 어떻게 받아야 하는지, 화면에는 어떻게 보여줄지 전혀 관여하지 않습니다.
따라서 기술적인 부분에 집중할 수 있지만, 변화에 반응하는 능력(민첩성)은 떨어진다는 단점이 있습니다.

### 도메인 변경의 어려움
 Layered Architecture의 도메인은 모든 레이어에 분산되게 됩니다. 따라서 도메인을 변경하는것은 쉽지 않습니다. 예를 들어 '고객' 도메인을 변경하려면 프레젠테이션, 비즈니스, 서비스, 데이터베이스 등을 모두 변경해야 하는 상황이 발생할 수 있습니다.<br>
  이런 이유로 레이어드 아키텍처 스타일은 도메인 주도 설계 방식(DDD)와 잘 어울리지 않습니다.
최근 DDD와 헥사고날 아키텍처가 같이 떠오른 이유라고 생각합니다.

## Controller와 Service
이제 본격적으로 본 주제에 대해서 이야기하려 합니다. 저의 주관적인 의견이 많이 들어있습니다.

### Controller
Presentation layer인 Controller(+View)는 사용자의 입출력을 담당합니다.<br>
클라이언트로부터 보내온 json을 오브젝트로 파싱하고, 반대로 객체를 응답(response) 포멧으로 컨버팅하는 역할을 합니다. (비즈니스 측면의 검증이 아닌) 형식적인 입력값 검증(Validation)도 진행합니다.<br>
핵심 비즈니스 로직보다는 클라이언트와의 상호작용을 주목적으로 합니다. 비즈니스 로직이 어떻게 수행되는지 알 필요가 없죠.

### Service
Application layer인 Service는 비즈니스 로직을 수행합니다. 회원가입, 주문, 게시글 조회 등을 수행합니다.<br>
Controller와 반대로 사용자와의 상호작용은 전혀 관심이 없습니다. 화면에 데이터를 어떻게 출력하는지, request 데이터 포멧이 어떤지, 어떤 통신 프로토콜을 사용하는지는 알지못합니다.
단지 Presentation layer에서 데이터를 가져와 비즈니스 로직을 수행하고, 다시 반환합니다.

### Request, Response 객체의 의존성
보통 Controller에서는 ModelView를 반환하거나 자바 객체를 반환합니다.
Spring은 `view Resolver을` 이용해 view를 찾고, model 렌더링 후 HTML을 완성합니다.
또는 `HttpMessageConverter` 인터페이스를 구현한 `SourceHttpMessageConverter`, `MappingJacksonHttpMessageConverter` 등을 이용해 직렬화하기도 합니다.

아래부터는 json을 기준으로 설명하겠습니다.<br>
Spring이 제공하는 애노테이션을 사용하면 json 파싱 및 메시지 컨버팅을 쉽게 할 수 있습니다.
`@RequestBody`를 이용해 자바 객체로 역직렬화하거나 `@ResponseBody`를 이용해 json 형식으로 직렬화할 수 있습니다.
안전한 HTTP 통신을 위해서는 클라이언트와 서버가 어떤 데이터를 주고받을건지 사전에 잘 약속해야합니다.

보통 직렬화, 역직렬화를 담당하는 자바 객체를 별도로 이용합니다.(Map 등을 이용하는 방법도 있지만요)<br>
**이러한 객체를 Service가 의존해야할까요?**

## Example
새롭게 Spring boot 프로젝트를 생성하겠습니다.
[깃허브에서 프로젝트 보기](https://github.com/redcarrot1/presentation-dependency)<br>

**Dependencies**
- Spring boot : 3.2.3
- Java : 17
- Spring Web
- Spring Validation
- Spring data JPA
- Lombok
- H2 Database

User를 Entity로 선언하고, dao로 Spring data JPA를 사용합니다.
```java
@Entity
@Table(name = "users")
@Getter
public class User {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private int age;

    protected User() {
    }

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```
```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

API는 User을 저장하는 기능을 제공합니다.
```java
@PostMapping("[버전]/users")
@ResponseStatus(HttpStatus.CREATED)
private UserSaveResponse saveUser(@RequestBody UserSaveRequest request) {
    // ...
}
```

Request와 Response는 아래와 같습니다.
```java
@Getter
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@AllArgsConstructor
public class UserSaveRequest {

    @NotBlank
    @Length(min = 1, max = 20)
    private String name;

    @Range(min = 1, max = 150)
    private int age;
}
```
```java
@Getter
@AllArgsConstructor
public class UserSaveResponse {

    private String name;
}
```


## V1
Contoller에서 넘어온 `UserSaveRequest`를 그대로 service에게 인자로 넘겨줍니다.
또한 service에서는 `UserSaveResponse`를 만들어서 반환합니다.
```java
@PostMapping("v1/users")
@ResponseStatus(HttpStatus.CREATED)
private UserSaveResponse saveUser(@RequestBody UserSaveRequest request) {
    return userService.save(request);
}
```
```java
public UserSaveResponse save(UserSaveRequest request) {
    User user = new User(request.getName(), request.getAge());
    userRepository.save(user);

    return new UserSaveResponse(user.getName());
}
```

어떤가요? 편리하기 때문에 이런 방법을 많이 사용하기도 합니다.<br>
하지만 중요한 문제가 있습니다. service가 request, response 형식을 의존하게 됩니다. 필드명이 바뀌면 어떤가요? 다른 API에서 해당 메서드를 사용하게되면 어떤가요? Presentation layer의 스팩은 언제든지 바뀔 수 있습니다. API 정의에 따라 다르기 때문입니다.<br>
레이어 격리의 문제도 생깁니다. Presentation layer가 변경되면 service 클래스도 변경해야합니다.<br>
Request, Response 클래스의 역할에도 문제가 있습니다. 해당 클래스에는 Bean Validation을 사용하기 위해 `@NotBlank` 등의 애노테이션이 붙어있습니다. Service 클래스에서 메소드 인자로 해당 애노테이션이 붙은 객체를 받을 특별한 이유는 없습니다.

## V2
```java
public record UserSaveDto(String name, int age) {
}
```
```java
@PostMapping("v2/users")
@ResponseStatus(HttpStatus.CREATED)
private UserSaveResponse saveUser(@RequestBody UserSaveRequest request) {
    UserSaveDto userSaveDto = new UserSaveDto(request.getName(), request.getAge());
    User user = userService.save(userSaveDto);

    return UserSaveResponse.from(user);
}
```
```java
public User save(UserSaveDto userSaveDto) {
    User user = new User(userSaveDto.name(), userSaveDto.age());
    return userRepository.save(user);
}
```

API 입출력 의존성으로부터 Service를 분리하였습니다.
대신 `record`를 이용하여 DTO로 인자를 전달했습니다. 인자 개수가 적으면(대략 3개 이하) 굳이 DTO로 감싸지 않고 필드를 넘겨주는 방법도 있습니다. 아예 컨트롤러에서 Entity를 조립하여 넘기는 방법도 있습니다.

Service는 더 이상 response 스팩에 맞춰서 반환하지 않습니다. 대신 더 많은 클래스와 API가 사용할 수 있는 객체를 반환하고 있습니다.(Entity나 DTO)

response에 정적 팩토리 클래스를 만들어서 API 스팩에 맞게 변환을 합니다. response가 entity를 의존하지만 괜찮습니다. 의존성은 (변하는 것 → 변하지 않는 것) 방향이 올바르기 때문입니다.


저는 여기서 의문이 있었습니다.
> Controller에 Entity가 노출되도 괜찮은가요?

이 질문의 근거가 어디서 출발했나 생각해봤습니다. '클라이언트에게 entity가 노출되면 안된다.' 라는 말을 잘못 이해한 것 같습니다. controller에서 반환 타입으로 entity를 사용하면 안된다는 것이지, controller에서 entity를 다루는 건 괜찮습니다.([근거](https://www.inflearn.com/questions/293399/service-%EC%9D%98-%EB%B0%98%ED%99%98%EA%B0%92%EC%9D%B4-dto%EC%9D%B8%EC%A7%80-entity-%EC%9D%B8%EC%A7%80%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EA%B6%81%EA%B8%88%ED%95%9C%EC%A0%90%EC%9D%B4-%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4))

---
DTO의 패키지는 어디에 두어야 하는지, DTO와 entity의 변환은 어디에서 하는지 등 아직 해결되지 않은 의문이 있습니다. 아마 많은 사람들이 고민하는 의문이지 않을까 생각됩니다.

DTO, entity와 관련된 질문과 김영한님의 의견을 모아봤습니다.

[DTO 변환 방법](https://www.inflearn.com/questions/15292/dto-%EB%B3%80%ED%99%98-%EC%8B%9C-%EC%9A%B0%EC%95%84%ED%95%9C%ED%98%95%EC%A0%9C%EB%93%A4%EC%9D%80-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%B2%98%EB%A6%AC%ED%95%98%EC%8B%9C%EB%82%98%EC%9A%94) <br>
[Service 레이어의 인자는 DTO? Entity?](https://www.inflearn.com/questions/53023/dto%EC%9D%98-layer%EC%97%90%EB%8C%80%ED%95%B4-%EC%A7%88%EB%AC%B8-%EB%93%9C%EB%A6%BD%EB%8B%88%EB%8B%A4) <br>
[DTO ↔️ Entity의 변환 위치](https://www.inflearn.com/questions/222914/dto-entity-%EB%B3%80%ED%99%98%EC%9D%80-%EC%96%B4%EB%8A%90-%EA%B3%B3%EC%9D%B4-%EC%A0%81%EC%A0%88%ED%95%A0%EA%B9%8C%EC%9A%94)

## 마치며
특히 최근 DDD가 유행처럼 번지고 있습니다.(자꾸만 높아지는 신입의 벽) 비즈니스 로직을 Service에 담아야할지, 도메인에 담아야 할지 고민입니다. 그 경계는 어디일까요.<br>
JPA+DDD의 조합은 어려운 것 같습니다. DB table이 중심이 되어 entity, domain을 구성하기 쉽습니다. 심지어 entity와 domain을 같은 것으로 생각하기도 합니다. 서비스를 구상할 때 domain을 떠올리지 않고, DB table을 떠올리는 저 자신을 보며.. 아직 멀었구나 생각이 듭니다.

(최근 이슈 중 하나로) JPA를 사용하면서 객체 연관관계를 사용하지 않고, 그냥 PK 타입을 사용하는 경우가 있다고 합니다. 테스트 클래스에서 `@Transactional`을 사용하면 된다, 안된다..<br>
그래도 이런 고민들이 논의되고 공유되면서 소프트웨어가 발전한다고 생각합니다.
