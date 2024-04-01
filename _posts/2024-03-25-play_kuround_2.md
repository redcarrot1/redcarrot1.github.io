---
title: "[playkuround] 행사 후기와 v2.0.4 버전 업데이트"
date: 2024-03-25 10:00:00 +09:00
categories: [playkuround]
tags:
  [
    version update
  ]
img_path: /assets/img/playkuround/update2/
---

## Intro
이번 게시물에선 오프라인 행사 후기와 playkuround server 업데이트 내역에 대해 포스팅하겠습니다.

## 오프라인 행사
[지난 게시물](https://redcarrot1.github.io/posts/email-asynchronous/)에 작성했던 것처럼 지난 주에 오프라인 행사를 진행했습니다.

![](1.png){: width="900" height="" }
_오프라인 행사 기획서와 당일 인스타 홍보 스토리_

띠부씰 60개와 뽑기 꽝 상품 100개를 준비했었습니다. 앱 다운로드나 쿠라운드 인스타 계정 팔로우와 스토리 태그를 하면 랜덤 뽑기를 할 수 있었어요.<br>
많은 분들이 찾아와주셨고, 그 결과 띠부씰 60개를 모두 소진했습니다. 찾아와주신 모든 분들과 추운 날씨에도 행사를 진행해준 PM에게 감사드립니다.

## 서버 업데이트
### iOS 대비 API 수정
2학기 개강 날에 출시를 목표로 현재 iOS를 제작하고 있습니다. 이에 따라 수정이 필요한 API가 있습니다.<br>
현재 저희 앱은 처음 가동 시, 앱 버전 체크 API를 호출합니다. 심각한 버그나 기능 수정으로 강제 업데이트가 필요한 경우 이를 강제할 수 있는 장치입니다.<br>
기존에는 안드로이드 앱만 존재했기 때문에 설치된 앱 버전만 쿼리 파라미터로 넣어줬었습니다.<br>
iOS가 개발되면 두 버전은 분리되어 운영되어야 하므로, 쿼리 파라미터에 OS 정보를 넣도록 수정했습니다.

- 기존: `/api/users/notification?version={version}`
- 수정: `/api/users/notification?version={version}&os={os}`

OS별 최신 업데이트 버전은 enum으로 관리했습니다.
```java
public enum AppVersion {

    ANDROID("2.0.2"),
    IOS("2.0.0");

    private static final Map<String, AppVersion> stringToEnum =
            Stream.of(values())
                    .collect(toMap(Object::toString, e -> e));
    private String latest_updated_version;

    AppVersion(String latest_updated_version) {
        this.latest_updated_version = latest_updated_version;
    }

    public static boolean isLatestUpdatedVersion(String os, String version) {
        // 생략
    }

    public static void changeLatestUpdatedVersion(String os, String version) {
        // 생략
    }
}
```

### user notification 수정
저희 DB은 유저별로 `notification` 필드를 가지고 있습니다. 이 필드를 이용해 개인별로 메시지를 보내거나 새로운 뱃지 추가 알림을 보낼 수 있습니다.<br>
메시지를 여러 개 저장할 필요도 있기 때문에 `{name1}#{description1}@{name2}#{description2}` 와 같이 특수문자(#, @)를 이용해 저장합니다.<br>

하지만 위의 방식을 사용하면 제 1정규형(1NF)를 어기게 됩니다.
> 1NF: 릴레이션에 속하는 속성의 속성 값이 모두 원자값(Atomic Value)만으로 구성되어야 한다.

User와 Notification은 1:N 관계이기 때문에 테이블을 분리하여 FK로 연결하는게 가장 일반적인 해법일 것 같습니다.<br>
하지만 저희 서비스는 제한된 컴퓨팅 환경을 사용하고 있습니다. 쿼리 하나가 매우 소중한 작업입니다.<br>
최대한 테이블 개수와 쿼리 개수를 줄이려고 시도하다보니 이런 상황이 만들어진 것 같습니다.<br>

실제로 작년 해커톤 때 비슷한 아이디어를 사용하기도 했습니다.<br>
사용자들이 음식 레시피를 정해진 템플릿을 이용하여 다른 유저들과 공유할 수 있는 플랫폼을 만들었었습니다.<br>
레시피에는 조리도구를 지정해야 합니다. 또한 조리도구로 레시피를 필터링하는 검색 기능도 제공해야 했습니다. 이때 사용한 방법은 **비트필드**입니다. 각 조리도구는 고유한 비트 필드를 갖습니다. 레시피에서 조리도구를 조회하거나, 조리도구로 레시피를 필터링하기 위해서는 AND 비트 오퍼레이션을 수행합니다. 과거의 저를 말리고싶네요.<br>
비트 오퍼레이션 함수 때문에 쿼리 작성에 문제가 있었습니다. 당시의 트러블슈팅을 [저희 팀원분](https://github.com/donghoony)이 [블로그 포스팅](https://blog.hoony.me/2023/08/19/use-bitwise-operation-on-jpa/) 했습니다.

하지만 (놀랍게도) 이번 업데이트에서는 테이블 분리를 하지 않았습니다. 대신 더 안정적으로 기능이 작동하도록 JPA의 `@Convert` 기능을 사용했습니다. Set을 이용한 이유는 중복 제거를 위해서입니다.
```java
@Entity
public class User extends BaseTimeEntity {
    //.. 생략
    @Convert(converter = NotificationConverter.class)
    private Set<Notification> notification;

    //.. 생략
```

```java
@Converter
public class NotificationConverter implements AttributeConverter<Set<Notification>, String> {

    @Override
    public String convertToDatabaseColumn(Set<Notification> notifications) {
        if (notifications == null) {
            return null;
        }

        StringBuilder sb = new StringBuilder();
        for (Notification notification : notifications) {
            if (!sb.isEmpty()) {
                sb.append("@");
            }
            sb.append(notification.getName())
                    .append("#")
                    .append(notification.getDescription());
        }
        return sb.toString();
    }

    @Override
    public Set<Notification> convertToEntityAttribute(String notifications) {
        if (notifications == null || notifications.isEmpty()) {
            return new HashSet<>();
        }
        return Arrays.stream(notifications.split("@"))
                .map(notification -> notification.split("#"))
                .filter(nameAndDescription -> nameAndDescription.length == 2)
                .map(nameAndDescription -> {
                    Optional<NotificationEnum> notificationEnum = NotificationEnum.fromString(nameAndDescription[0]);
                    return notificationEnum
                            .map(anEnum -> new Notification(anEnum, nameAndDescription[1]))
                            .orElse(null);
                })
                .filter(Objects::nonNull)
                .collect(Collectors.toSet());
    }
}
```

테이블을 분리하려면 DB 스키마가 변경되어야 합니다.
현재는 플레이쿠라운드가 활발하게 서비스 중이어서 살짝 부담스럽게 느껴집니다. 지금은 DB가 싱글로 실행 중이기도 하고..<br>
빠른 시일 내에 테이블 분리하는 작업을 수행하도록 하겠습니다.

## Outro
처음 만들었을 때는 그게 완성형인 줄 알았습니다. 그런데 운영하다 보니 유지 보수할 것들이 많이 생기는 것 같습니다.(다행히 버그 제보는 안 들어오네요)<br>
저희 서비스는 달(month)가 넘어갈 때 랭킹 시스템이 초기화되고, 랭킹 1~3등에게 새로운 뱃지가 추가됩니다. 현재는 해당 기능이 자동화되어 있지 않아요.<br>
Spring batch를 이용하여 자동화시키는 게 숙제입니다. notification 테이블도 분리하고..