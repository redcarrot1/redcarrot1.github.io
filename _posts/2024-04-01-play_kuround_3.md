---
title: "[playkuround] v2.0.5 버전 업데이트"
date: 2024-04-01 10:00:00 +09:00
categories: [playkuround]
tags:
  [
    version update 
  ]
---

## Intro
이번 게시물에선 새롭게 업데이트된 [playkuround v2.0.5](https://github.com/playkuround/playkuround-server/releases/tag/v2.0.5)을 소개하겠습니다.

## 이메일 인증 버그 수정
플레이쿠라운드는 회원 인증이 필수입니다. 건국대생을 타겟으로 만들었기 때문에 `@konkuk.ac.kr` 도메인으로만 이메일 인증이 가능합니다.<br>
모바일 애플리케이션에서 서버로 인증 메일 주소를 보내면, 서버에서 메일은 전송합니다. 이때 이메일 전송 횟수를 DB에 저장하여, 무차별적으로 메일 전송하는 것을 방지했습니다.<br>

하지만 간과했던 사실이 있었으니.. **일반적으로 이메일 주소는 대소문자를 구분하지 않습니다.** ([참고](https://support.google.com/mail/thread/242316772?hl=ko&msgid=242432907)) 테스트해보니 건국대 이메일도 대소문자를 구분하지 않았습니다.<br>
따라서 길이가 n인 아이디를 사용하는 경우, 2^n 개의 이메일 주소를 만들 수 있습니다. 저희 서비스 정책상 이메일 전송 회수를 5회로 제한하고 있으니, 5*(2^n)번 전송이 가능합니다.<br>
위 버그를 수정하기 위해 presentation 레이어에서 business 레이어로 데이터를 넘길 때, 항상 lowerCase로 변환하도록 수정했습니다.

```java
 public ApiResponse<AuthVerifyEmailResponse> authEmailVerify(@RequestParam("code") String code, @RequestParam("email") String email) {
    AuthVerifyEmailResult authVerifyEmailResult = authEmailVerifyService.verifyAuthEmail(code, email.toLowerCase());
    return ApiUtils.success(AuthVerifyEmailResponse.from(authVerifyEmailResult));
}
```

## 월 업데이트 자동화
플레이쿠라운드는 매월 1일에 (비교적 큰) 업데이트를 수행합니다. <br>

1. 토탈 스코어 랭킹 1, 2, 3위 뱃지 부여
2. 랜드마크별 랭킹 초기화
3. 유저별 최고 스코어(+랭킹) 업데이트
4. 종합 랭킹 초기화

이번 4월 1일 새벽 12시에 위 작업을 했습니다. 무려 **수동으로** 말입니다.<br>
이번에 직접 DB에 데이터를 수정해가며 업데이트를 했는데요, 문제가 있습니다.
1. 시간이 오래걸린다. (작업 도중에는 서비스는 멈춥니다.)
2. 휴먼 에러가 발생할 수 있다. (가장 문제)

이번에는 대략 15분 정도 걸린 것 같습니다. 저희 서비스 특성상 새벽 시간대에는 이용률이 매우 떨어지지만, 서비스를 중지하고 작업한다는 것은 개선해야 할 사항입니다. 특히 직접 값 하나하나를 처리하다 보니 사람의 실수가 있을 수 있습니다.

따라서 위 작업을 한 번에 수행해주는 API를 만들었습니다.<br>
왜 배치 프로세스로 하지 않았냐면.. 관련 지식이 아직 부족합니다.<br>
일단은 한 달에 한 번 호출되는 서비스니 수동 호출로 만들었습니다. 추후에 더 공부한 후에 배치 작업으로 업데이트하겠습니다.

## Outro
꾸준히 서비스를 이용해주시는 분들이 많습니다. DB에 데이터가 들어올 때마다 기분이 좋네요.<br>
(정상 접근에 한해) 과도한 트래픽으로 인해 서비스가 멈추면 슬프면서 동시에 기쁘다고 합니다. 그만큼 많은 유저 트래픽이 발생한 것이니깐요.<br>
추가로 이번 축제 때 오프라인 행사를 준비하고 있습니다. 많은 관심 부탁드립니다.