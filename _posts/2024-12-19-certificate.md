---
title: "Certificate"
date: 2024-12-19 10:00:00 +09:00
categories: [암호학]
tags:
  [
     Certificate, SA
  ]
math: true
img_path: /assets/img/cryptography/
---
## Man in the Middle of Attack
- 공개키 신뢰 문제

![](16.png){: width="500" height="" }

- Bob은 $P_A$가 Alice의 것임을 어떻게 믿을 수 있을까?
- 공개키에 대한 증명(Certification) 필요

## 공개키 기반구조(PKI)
- 인증서(Certificate): 각 사용자의 공개키에 대한 증명서
- 인증 기관(CA: Certificate Authority): 각 사용자의 공개키에 대한 인증 및 증명서 발급 기관
    - Trusted third party
- **PKI(Public-Key Infrastructure)**
    - **사용자 + 공개키 + 인증서 + 인증기관**으로 구성

## Hierarchy of Certificate Authorities
![](17.jpg){: width="300" height="" }
- PAA(Policy Approval Authorities): 정책승인기관
- PCA(Policy Certification Authorities): 정책인증기관
    - KISA(Korea Information Security Agency, 한국인터넷진흥원)
- **Root CA**(Root Certification Authoriy): 인증서를 발급하는 최상위 기관
    - Root cerificate 관리 (Root certificates are self-signed)
    - [KISA RootCA](https://www.rootca.or.kr/kor/accredited/accredited03_01View.jsp?seqno=1)
    - [NAVER Cloud RootCA](https://navercloudtrust.com/)
- **CA**(Certification Authoriy): 인증기관
- RA (Registration Authority): 등록기관

## 인증서 구성
- The format of these certificates is specified by the **X.509**
- 데이터 영역: 기본영역 + 확장 영역
- 전자서명: 데이터 영역을 발급자의 개인키로 전자서명
    - 인증서 무결성, 인증 제공

![](18.png){: width="400" height="" }

- 버전: 대부분 버전 3
- 일련번호: 인증서 발급한 인증기관내의 인증서 일련번호
- 서명 알고리즘: 인증서를 발행 시 사용한 서명 알고리즘
    - ex. sha1RSA
- 발급자
    - 인증서 발급 인증기관의 DN(Distinguish Name)
    - X.500 표준에 따라 명명된 이름
    - C(Country), O(Organization), OU(Organization Unit), CN(Common Name) 필드로 구성    
- 유효기간(시작, 끝)
    - 인증서를 사용할 수 있는 기간
    - 시작일과 만료일을 기록(초 단위까지)
- **주체**
    - 인증서 소유자의 DN
    - **인증서 소유자의 공개키**

## 인증서의 기능

- **누구나 사용자의 인증서를 획득 가능**
    - 인증서의 데이터 영역에서 공개키 획득 가능
- **인증기관 이외에는 인증서를 수정/발급 불가**
    - 인증서 자체에도 인증기관의 전자서명이 되어 있다.
    - CA는 X.509 표준에 따라 인증서 폐기 목록(CRL; Certificate Revocation List)을 관리해야 함

<br>

## 인증서 사용

![](19.png){: width="300" height="" }

- 기본 구성: 사용자 인증서 + CA 인증서 + RootCA 인증서(또는 공개키)
- 사용자 인증서
    - 개인 사용자 공개키에 대한 인증
    - CA의 개인키로 전자서명
    - CA의 공개키를 검증하기 위해 Root CA의 전자서명을 사용한다.
- CA의 전자서명 인증
    - CA의 공개키에 대한 인증
    - Root CA가 CA의 공개키에 대한 인증서 발급
- Root CA 인증서는 누가 인증해주나요?
    - 일반적으로 Root CA의 인증서는 운영체제나 브라우저에 탑재되어 있음
    - 크롬 인증서 관리자 : 'chrome://certificate-manager/' 에서 확인 가능
    - macOS : '키체인 접근 → 시스템루트 → 인증서' 에서 확인 가능

### 인증서 검증 순서

Notation: Verify(signature verify key, target certificate)

1. Verify(Root CA 공개키, CA 인증서) : CA의 인증서는 올바르다. 인증서에서 CA 공개키 꺼내기
2. Verify(CA 공개키, 사용자 인증서) : 사용자 인증서는 올바르다. 인증서에서 사용자 공개키 꺼내기
3. Verify(사용자 공개키, 사용자 전자서명)

<br>

## 네이버 클라우드 Root CA
[공식 블로그 포스팅](https://blog.naver.com/PostView.naver?blogId=n_cloudplatform&logNo=222942986230)
- 2022년에 글로벌 주요 OS, 브라우저에 RootCA 인증서 탑재 완료
  - 애플 ios, macOS, iPadOS, WatchOS, tvOS
  - 구글 Android, Chrome
  - MS OS, Mozila Firefox, Naver Whale
- macOS에서 확인하기
  - '키체인 접근 → 시스템루트 → 인증서'에서 확인 가능
  
![](20.png){: width="400" height="" }