---
title: Zero Knowledge
date: 2024-01-11 10:00:00 +09:00
categories: [암호학]
tags:
  [
    암호학, cryptography
  ]
math: true
img_path: /assets/img/cryptography/
---

## Zero Knowledge Proof
- 이론적으로 완벽을 만들기 위함
    - 실용적이진 않음(실제로 사용은 안함)
    - RSA의 경우 NP-Complete가 아니지만 실용적이기 때문에 사용
- $P$(증명하고 싶어하는 사람) , $V$(확인하려는 사람)
- $P$는 자신이 비밀정보 $X$를 안다는 것을 $V$에게 증명하려고 한다.
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
        - 가짜 사이트, 가짜 프롬프트, 카드 리더기 복사.. 등의 문제가 발생함

<br>

## Zero Knowledge Proof of Graph Isomorphism

- Graph Isomorphism Problem : 그래프의 모양이 동형이냐?
    - 노드 번호를 잘 rename하면 완전히 두 그래프를 같게 할 수 있냐?
    - NP-Complete 문제이다.
        - node $n$개, edge $m$개
        - rename 방법: $n!$

<br>

## Details of the Proof
- 공개된 정보 : $G_1, G_2$ + $G_1, G_2$ is isomorphic
- Secret Info$(X)$ : mapping between $G_1$ and $G_2$
- $G_1, G_2$를 만드는건 쉽다. (예를 들어 랜덤하게 $G_1$을 만들고 permutation시켜 $G_2$를 만든다.)
- $P$가 $G_1, G_2$를 만들고, ‘그 매핑을 아는 사람은 나다’는 것을 증명
- A Round
    1. $P$는 $G_1$을 rename해서 $G_{temp}$를 만들어 $V$에게 전달
        - $P$는 $G_1\leftrightarrow G_2 \leftrightarrow G_{temp} \leftrightarrow G_1$ 매핑을 모두 안다.(자신이 $G_1, G_2, G_{temp}$를 만들었기 때문)
    2. $V$는 $G_1\sim G_{temp}$ 또는 $G_2 \sim G_{temp}$ 의 매핑을 물어본다.
    3. $P$는 mapping을 증명해준다.
        - 가짜 $P'$은 높은 확률로 여기서 실패
- 가짜 $P'$은 $G_1 \leftrightarrow G_2$ 매핑을 모르기 때문에 $G_1\sim G_{temp}$ 와 $G_2 \sim G_{temp}$ 중 최대 1개만 알 수 있다.
    - 둘 다 안다는 건 사실상 $G_1 \leftrightarrow G_2$ 을 아는거나 마찬가지이기 때문
- Round를 여러번 진행하면 $V$는 $P$를 인정

<br>

## Why is it Zero Knowledge?

- 가장 중요한 핵심: no transfer
- $V$가 그래프 매핑을 계산할 수 있는 경우도 있다.
    - Round 1
        - $P \rightarrow V : G_{temp~1}$
        - $V \rightarrow P : G_1 \sim G_{temp~ 1}$  mapping은 어떤가요?
        - $P \rightarrow V:G_1 \sim G_{temp~1}$ mapping을 알려줌
    - Round 2
        - $P \rightarrow V : G_{temp~2}$
            - **이때 $G_{temp~1}=G_{temp~2}$ 이라면?**
        - $V \rightarrow P : G_2 \sim G_{temp~ 2}$  mapping은 어떤가요?
        - $P \rightarrow V:G_2 \sim G_{temp~2}$ mapping을 알려줌
        - $V$는 $G_1, G_2$의 mapping을 계산 할 수 있다.
- Round 개수를 $n!$ 번 하면 분명 $G_{temp}$는 서로 겹치거나, $G_1 또는 G_2$와 겹치는 경우가 발생한다.
    - Round를 진행할 때마다 아주 조금씩 정보가 전달되는게 아닌가?
    - **정보가 전달되는게 아니라, $V$가 계산하는거다.**
- **$V$가 실제 $P$와 통신하는게 아니라, 자기 스스로 fake $P$를 만들어서 대화한것(Transcript)을 구별할 수 없다.**
    - 실제 $P$와 통신해서 만들어진 Transcript와 $V$ 혼자 만든 Transcript가 등장할 확률이 동일하다.
- Why not used?
    - 실용적이지 않음

<br>

## Practical (using RSA) (실제 사용)

- Zero Knowledge라는 증명이 없음. 아주 많이 통신하다보면 $d$가 노출될수도
- 나(A)와 은행(Bank)와 거래
- 나 : $n, e$는 공개, $p, q, d$는 비밀정보
- 은행 : 너가 $d$를 알고있다는 것을 증명하라
    - $d$를 은행한테 보내주는게 가장 쉽지만 위험함
- 방법
    1. 은행이 Random message m을 만들어서 나에게 전송
    2. 나는 사인을 해서 $m^d$를 다시 은행한테 전송
    3. 은행은 공개키 $e$를 이용해 signature verification

<br>

## Blind Signature(실제 사용)
- Massey-Omura와 유사하다
- B가 메시지 $m$을 숨기고 싶을 때 사용

1. B가 A한테 메시지 $m$에 대해 사인 요청
    - 이때 B가 $m$을 자기 공개키로 암호를 걸어 보냄 ($m^{e_b}$  전송)
    - 즉, A는 메시지 m을 모른 상태로 사인
2. A가 사인을 해서 다시 B에게 보냄 : $(m^{e_b})^{d_a}$
3. B는 받아서 자기 비밀키로 복구 : $((m^{e_b})^{d_a})^{d_b}=m^{d_a}$