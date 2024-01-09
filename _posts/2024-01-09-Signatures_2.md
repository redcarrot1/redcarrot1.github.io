---
title: Signatures 2
date: 2024-01-09 10:00:00 +09:00
categories: [암호학]
tags:
  [
    암호학, cryptography
  ]
math: true
img_path: /assets/img/cryptography/
---

## Cryptographic Hashing
- RSA Signature은 오리지널 문서와 크기가 같다. (너무 큼)
- Signature가 문서 사이즈에 의존하지 않고, 적절한 고정된 길이를 사용하도록 하자 → **Hashing!**

- 메시지보다 해쉬가 더 작기 때문에, 겹치는 해쉬값이 존재한다.
    - 해쉬를 checksum과 같은 걸 사용하면 조작하기가 매우 쉬움
    - 즉, 똑같은 해쉬값을 찾는게 어려운 해슁을 사용해야 한다.
    - 이렇게 나온 해쉬값은
        1. 원본 메시지가 바뀌면 해쉬값이 바뀌어야 하고
        2. 예측하기 매우 어려워야 한다.
- 가장 많이 사용하는 해싱 알고리즘은 SHA(Secure Hash Algorithm)

<br>

## birthday Paradox
- $n$명이 함께 있을 때, 모두 다른 생일을 가질 확률은 생각보다 낮다.
- 하지만 $n$명 중에 '나'와 같은 생일자가 없을 확률은 생각보다 크다.

### Meaning
- The following two problems are quite different
- Find two message $M_1$ and $M_2$ such that $h(M_1 )=h(M_2)$
- Given a message $M_1$, find another message $M_2$ such that $h(M_1 )=h(M_2)$
- 결론: original message을 hasing한 결과와 동일한 hasing값을 갖는 another message를 찾는 것은 매우 힘들다.

<br>

## Typical Usage (최종판)
- RSA encrypts only the Key for AES
- AES encrypts the Message
- SHA hashes the Message
- RSA makes Signature with Hash Value
![](5.png)

<br>

## Massey-Omura 

- 공개된 정보, 공유하는 정보가 아무것도 없는 암호
    - state를 유지해야하기 때문에 실용성이 낮다.
- Suitcase Example
    - A가 B에게 안전하게 가방을 보내고 싶다.
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

## Cryptographic Protocols

- 네트워크 프로토콜과 다른 점
    - 네트워크 프로토콜은 반대편이 잘못보내면 통신이 안된다.
    - 암호 프로토콜은 누가 속일려고 하는 상황까지 고려해야 한다.
- 암호 프로토콜 사용 예시 : Bank Transactions, Coin Toss, Internet Poker

### example: Secret Sharing
- Setting: $n$명이 사람이 비밀값 $s$를 공유하고자 한다.
- 목적: $k$명이 모이면 $s$를 계산해 낼 수 있지만, $k-1$명이 모이면 그 어떤 정보도 알 수 없어야 한다.
- How?
    - 신뢰되는 사람들(기관들)이 필요(약점)
    - 기본 전제 : $\mod p$
    1. 식 생성 : $f(x)=a_{k-1}x^{k-1}+a_{k-2}x^{k-2}+\cdots+a_{1}x^{1}+s$ 
        - 신뢰되는 사람이 만든다. ($a_i$ is random)
    2. $f(i)$를 $i$번째 사람에게 전달한다. → 점 $(i, f(i))$
    3. 점 $k$개가 있으면 $f(x)$를 구할 수 있다. → $s$를 구할 수 있음
        - 하지만 $k-1$개의 점이 있으면 $s$를 전혀 알 수 없다.
