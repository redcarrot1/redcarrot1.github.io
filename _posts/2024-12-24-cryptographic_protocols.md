---
title: "Cryptographic Protocols"
date: 2024-12-24 10:00:00 +09:00
categories: [암호학]
tags:
  [
     암호학, cryptography
  ]
math: true
img_path: /assets/img/cryptography/
---

본 포스팅에서는 암호를 이용한 여러 프로토콜을 살펴보고자 한다. 사실 각 주제마다 방대한 양이지만 여기서는 가볍게 컨셉만 확인하자

## Massey-Omura

- 공개 또는 서로 공유하는 정보가 아무것도 없는 암호
  - 하지만 state를 유지해야하기 때문에 실용성이 낮다.
- Suitcase Example
  - Goal: A가 B에게 안전하게 가방을 보내고 싶다.
  - A가 가방에 자신의 자물쇠로 잠금
  - B는 가방에 자신의 자물쇠로 다시 잠가서 A에게 보냄
  - A는 자신의 자물쇠를 열고 B에게 다시 보냄
  - B는 자신의 자물쇠로 가방을 연다.
- Detail
  - Setting
    - Shared $p$
    - A's Secret: $e_a, d_a, e_a\times d_a=1 \mod {p-1}$
    - B's Secret: $e_b, d_b, e_b\times d_b=1 \mod {p-1}$
  - A → B : $m^{e_a} \mod p$
  - B → A : $(m^{e_a})^{e_b} = m^{e_ae_b} \mod p$
  - A → B : $(m^{e_ae_b})^{d_a} = m^{e_b} \mod p$
  - B : $(m^{e_b})^{d_b}=m \mod p$

<br>

## Secret Sharing

- 하나의 비밀정보를 다수의 비밀조각으로 분할하여 다수의 신뢰할 수 있는 사람들에게 공유
- 예시 알고리즘: $(k,n)$-threshold scheme (또는 Shamir’s Secret Sharing)
    - $n$개의 비밀조각 중에서 $k$개 이상의 비밀조각만 있으면 원래의 비밀 정보를 복원할 수 있는 시스템
    - $k-1$차 다항식 이용
- How?

  1. 식 생성 : $f(x)=a_{k-1}x^{k-1}+a_{k-2}x^{k-2}+\cdots+a_{1}x^{1}+S$ 
     - $a_i$ is random
  2. 서로 다른 $(i, f(i))$을 $n$개 만들어 배분
  3. 점 $k$개가 있으면 $f(x)$를 구할 수 있다. → $S$를 구할 수 있음
     - 연립방정식 또는 Lagrange 보간법 이용
     - 하지만 $k-1$개의 점이 있으면 $S$를 전혀 알 수 없다.

### 예시: (3, 6) threshold scheme
- 정보를 6개의 비밀 조각으로 나눔, 3개 이상의 비밀 조각이 모이면 정보 획득 가능
- 비밀 조각 만들기
    - $S = 1234, a_1 = 166, a_2 = 94$
    - $f(x) = 94x^2+166x+1234$
    - $s_0 = (1, 1494), s_1= (2, 1942), s_2 = (3, 2578), s_3= (4,3402), s_4 = (5,4414), s_5 = (6,5614)$
- 비밀복원(Reconstruction)
    - 3개 이상의 조각이 있으면 $f(x)$을 만들 수 있다.

<br>

## Bit Commitment

### Fair Coin Flip
> 온라인 상에서 동전 던진기

- 문제
    - Alice와 Bob이 전화상으로 통화를 하고 있다
    - Alice가 동전을 던진다. 결과는 앞면(head) 또는 뒷면(tail)
    - Bob이 앞면인지 뒷면인지 추측한다 (guessing)
    - Alice가 정답을 알려준다
    - Bob이 Alice가 정답을 얘기했음을 확인한다.
- Bob은 Alice가 동전을 던지는 것을 눈으로 직접 확인하지 않고, Alice가 동전을 던져 나온 결과를 어떻게 믿을 수 있을까?
    - **Bit Commitment 필요!**

### Coin Flipping using One-Way Functions

- Alice chooses a random number $x$
- Alice computes $y = f(x)$
    - 예를 들어 $f(x)$는 Hash(x)
    - 사실 종류도 많고 방법도 다양하다.
- Alice sends $y$ to Bob
- Bob guesses whether $x$ is even or odd, and sends his guess to Alice
- Alice announces the result of the coin flip, and sends $x$ to Bob
- Bob confirms that $y = f(x)$

- Bit Commitment는 온라인 상에서 안전한 데이터 송수신을 위해 기본이 되는 프로토콜 중 하나이다. 근데 목적이 암호화와는 조금 다르다. 두 사용자가 대화형으로 메시지를 주고받는 과정에서 서로 속이지 않는지 진위여부를 판단하기 위해 사용된다.

<br>

## Bitcoin
- 추가 예정

<br>

## Zero Knowledge Proof
- $P$(증명하고 싶어하는 사람) , $V$(확인하려는 사람)
- **$P$는 자신이 비밀정보 $X$를 안다는 것을 $V$에게 증명하려고 한다.**
- 가장 쉬운 방법: $V$에게 $X$를 보여줘서 증명함
  - 문제점 : $V$에게 비밀정보를 넘겨줘야 한다.


- **이 프로토콜의 최종 목적**
  - $V$에게 $X$의 일부 정보라도 주고 싶지 않다.(**no transfer**)
  - '$P$가 $X$를 알고 있다'는 정보만 주고 싶지, $X$ 자체를 주고 싶지는 않다.
  - 또한 $X$를 모르는 제3자는 매우 높은 확률로 증명에 실패해야 한다.
- 왜 이 프로토콜을 사용할까?
  - 인증서로 로그인할 때 이미 어느정도 구현되어 있다.
  - 비밀번호 로그인은 구현이 안되어 있음
    - **내가 비밀번호를 안다는 것만 알려주고 싶지 비밀번호는 알려주고 싶지 않음**
    - 비밀번호를 안다는 것을 가장 쉽게 증명하는 방법은 비밀번호 자체를 넘겨주는 것
    - 하지만 가짜 사이트, 가짜 프롬프트, 카드 리더기 복사.. 등의 문제가 발생함
  - 이론적으로 완벽을 만들기 위함
    - 실용적이진 않음(실제로 사용은 안함)
    - RSA의 경우 NP-Complete가 아니지만 실용적이기 때문에 사용

### Zero Knowledge Proof of Graph Isomorphism

- Graph Isomorphism Problem : 두 그래프는 동형인가?
  - 노드 번호를 잘 rename하면 완전히 두 그래프를 같게 할 수 있냐?
  - NP-Complete 문제이다.
      - node $n$개, edge $m$개
      - rename 방법: $n!$

### Details of the Proof
- 공개된 정보 : $G_1, G_2$
  - 두 그래프 $G_1, G_2$는 동형임
- Secret Infomation$(X)$ : mapping between $G_1$ and $G_2$
  - 두 그래프 자체는 공개되어도 괜찮지만, 매핑 정보는 비공개.
- $G_1, G_2$를 만드는건 쉽다. (예를 들어 랜덤하게 $G_1$을 만들고 permutation시켜 $G_2$를 만든다.)
- $P$가 $G_1, G_2$를 만들고, ‘그 매핑을 아는 사람은 나다’는 것을 증명
- 대략적인 알고리즘
    1. $P$는 $G_1$을 rename해서 $G_{temp}$를 만들어 $V$에게 전달
        - $P$는 $G_1\leftrightarrow G_2 \leftrightarrow G_{temp}$ 매핑을 모두 안다.(자신이 $G_1, G_2, G_{temp}$를 만들었기 때문)
    2. $V$는 $G_1\sim G_{temp}$ 또는 $G_2 \sim G_{temp}$ 의 매핑을 물어본다.
    3. $P$는 mapping을 증명해준다.
        - 가짜 $P'$은 높은 확률로 여기서 실패
- 가짜 $P'$은 $G_1 \leftrightarrow G_2$ 매핑을 모르기 때문에 $G_1\sim G_{temp}$ 와 $G_2 \sim G_{temp}$ 중 최대 1개만 알 수 있다.
    - 둘 다 안다는 건 사실상 $G_1 \leftrightarrow G_2$ 을 아는거나 마찬가지이기 때문
- Round를 여러번 진행하면 $V$는 $P$가 $G_1$와 $G_2$ 사이의 매핑 정보를 정확히 알고있다고 인정

### Why is it Zero Knowledge?

- 가장 중요한 핵심: no transfer
- $V$가 그래프 매핑을 계산할 수 있는 경우도 있다.
    - Round 1
        - $P \rightarrow V : G_{temp~1}$
        - $V \rightarrow P : G_1 \sim G_{temp~ 1}$  mapping 정보는?
        - $P \rightarrow V:G_1 \sim G_{temp~1}$ mapping을 알려줌
    - Round 2
        - $P \rightarrow V : G_{temp~2}$
            - **이때 $G_{temp~1}=G_{temp~2}$ 이라면?**
        - $V \rightarrow P : G_2 \sim G_{temp~ 2}$  mapping 정보는?
        - $P \rightarrow V:G_2 \sim G_{temp~2}$ mapping을 알려줌
        - $V$는 $G_1, G_2$의 mapping을 계산 할 수 있다.
- Round 개수를 $n!$ 번 하면 분명 $G_{temp}$는 서로 겹치거나, $G_1$ 또는 $G_2$와 겹치는 경우가 발생한다.
    - Round를 진행할 때마다 아주 조금씩 정보가 전달되는게 아닌가?
    - **정보가 전달되는게 아니라, $V$가 계산하는거다.**
- **$V$가 실제 $P$와 통신하는게 아니라, 자기 스스로 fake $P$를 만들어서 대화한것(Transcript)을 구별할 수 없다.**
    - 실제 $P$와 통신해서 만들어진 Transcript와 $V$ 혼자 만든 Transcript가 등장할 확률이 동일하다.
- Why not used?
    - 실용적이지 않음


### 실제 사용하는 알고리즘 (using RSA)

- Zero Knowledge라는 증명은 없음
- 상황: 나(A)와 은행(Bank)와 거래
  - 나 : $n, e$는 공개 정보, $p, q, d$는 비밀정보
  - 은행 : 너가 $d$를 알고있다는 것을 증명하라
- $d$를 은행한테 직접 전송하지 않고, 자신이 $d$를 알고있다는 사실을 어떻게 증명할 것인가
- 방법
    1. 은행이 Random message $m$을 만들어서 나에게 전송
    2. 나는 사인을 해서 $m^d$를 다시 은행한테 전송
    3. 은행은 공개키 $e$를 이용해 signature verification
        - $m^{ed}==m$이라면 내가 $d$를 알고있다는 사실을 증명한 셈이다.

<br>

## Blind Signature
- Massey-Omura와 유사
- B가 A에게 메시지 $m$에 사인 요청. 단, $m$의 내용은 숨기고싶다.
- 구현 방법은 여러가지가 있음. 여기선 RSA를 응용한 버전 중 하나

1. B가 A한테 메시지 $m$에 대해 사인 요청
  - 이때 B가 $m$을 자기 공개키로 암호를 걸어 보냄 ($m^{e_b}$ 전송)
2. A가 사인을 해서 다시 B에게 보냄 : $(m^{e_b})^{d_a}$
  - A는 메시지 $m$을 모른 상태로 사인
3. B는 받아서 자기 비밀키로 복구 : $((m^{e_b})^{d_a})^{d_b}=m^{d_a}$
