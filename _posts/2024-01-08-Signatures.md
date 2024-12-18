---
title: Signatures
date: 2024-01-08 10:00:00 +09:00
categories: [암호학]
tags:
  [
    암호학, cryptography
  ]
math: true
img_path: /assets/img/cryptography/
---

## Signatures(전자서명)

- Signatures가 만족해야 하는 조건
    1. 나만 만들 수 있어야 한다.
    2. 모든 사람이 내 싸인임을 확인할 수 있어야 한다.
    3. **문서와 분리가 되면 안된다. - 문서마다 싸인을 다르면 문서끼리 떼다 붙일 수 없다.**
- **전자서명의 보안 요구사항**
    - **위조불가**(Unforgeable)
        - 서명자만이 전자 서명 생성할 수 있다.
    - **인증**(Authentication)
        - 전자 서명의 서명자를 확인할 수 있다.
    - **재사용 불가**(Not Reusable)
        - 한 번 생성된 서명을 다른 문서의 서명으로 사용 불가
    - **변경 불가**(Unalterable)
        - 서명된 문서의 내용 변경 불가
        - 데이터의 무결성(Integrity) 보장
    - **부인방지**(Non Repudiation)
        - 서명자가 나중에 서명한 사실을 부인할 수 없음
<br>

## RSA Signature

- Setting
    - $p$ and $q$ : large Primes,   $n=pq$,   $e$ : Encryption Key,   $d$ : Decryption Key
        - public: n, e
        - private: p, q, d
    - $ed=1 \mod {(p-1)(q-1)}$
    - $\varphi(n)=\varphi(pq)=(p-1)(q-1)$ (오일러 정리)
        - $pq-pq*\left(\frac 1p+\frac 1q\right)+1$
- Signature for Message $m$ (Only the Owner of Key)
    - $s(m) = m^d \mod n$
- Verifying Signature(Anyone)
    - Compute $v=\left( s(m) \right)^e \mod n$
    - 해독한 문자 $v$와 문서인 $m$이 서로 같은지 확인한다.
    - 누구나 확인할 수 있다.

<br>

## ElGamal Signature

- Setting
    - Public : $p, g$ (shared by all)
    - Alice’s Public Info : $y=g^a \mod p$
    - Alice’s Secret Info : $a$
- Signautre for Message $m$
    - Random $k$, $r=g^k \mod p$, $s=(m-ar)k^{-1} \mod{(p-1)}$
    - Signature is $(r,s)$
- Verifying Signature (Anyone)
    - Check if $g^m = y^rr^s \mod p$

### Signature Correctness

- $y^rr^s = (g^a)^r(g^k)^{(m-ar)k^{-1}}=g^{ar+m-ar}=g^m \mod p$

<br>

## DSA(Digital Signature Algorithm)
- For two primes: $p, q (q<p)$
- Key Generation
    - private key: choose $x$ at random, $0 < x < q$
    - public key: $y = g^x \mod p$
- Signing $M$
    - for a random $k$, $0 < k < q$
    - $r = (g^k \mod p) \mod q$
        - 단, $r ≠ 0$
    - $s = (k^{-1}(H(M) + xr)) \mod q$
    - Signature: $<M, r, s>$
- Verifying
    - $w = s^{-1} \mod q$
    - $u1 = H(M)w \mod q = H(M)/s \mod q$
    - $u2 = rw \mod q = r/s \mod q$

$$
v = ((g^{u1}y^{u2}) \mod p) \mod q
$$

$$
=((g^{H(M)/s}g^{x(r/s)}) \mod p) \mod q
$$

$$
= (g^{(H(M)+xr)/s} \mod p) \mod q
$$

$$
= (g^{\frac {(H(M)+xr)}{k^{-1}(H(M)+xr)}} \mod p) \mod q
$$

$$
= (g^k \mod p) \mod q
$$

- check if $v = r$

<br>

## Cryptographic Hashing

### 문제1: 오리지널 문서 크기가 커질수록 전자서명의 크기도 커진다.
- Signature가 문서 사이즈에 의존하지 않고, 적절한 고정된 길이를 사용하도록 하자 → **Hashing!**

### 문제2: 제 3자가 전자서명 생성 가능 (ex. RSA signature)
- 공격자가 두 개의 서명 $S_1$과 $S_2$를 알고 있을 때
    - $S_1 = M^d_1 \mod n, S_2 = M^d_2 \mod n$
- $(M1· M2)$에 대한 새로운 서명 $S'$ 생성 가능
    - $S' = S_1· S_2 \mod n = M_1^d · M_2^d \mod n = (M_1· M_2)^d \mod n$
    - 주인이 아님에도 전자성명을 만들 수 있는 문제가 발생
    - $(M1· M2)$는 의미가 없는 메시지이잖아요? → 그건 중요하지 않다. 어째든 다른 사람의 성명을 만든다는게 위험한거임
    - 어떻게 해결 가능할까? → **Hash!**

$$
S_1 = H(M_1)^d \mod n\\S_2 = H(M_2)^d \mod n
$$

$$
S_1S_2\mod n = (H(M_1)H(M_2))^d \mod n \\ \not = H(M_1M_2)^d \mod n
$$

### 문제3: 해시 충돌
- 메시지보다 해쉬가 더 작기 때문에, 겹치는 해쉬값이 존재한다.
    - 해쉬를 checksum과 같은 걸 사용하면 조작하기가 매우 쉬움
    - 즉, 똑같은 해쉬값을 찾는게 어려운 해슁을 사용해야 한다.
    - 이렇게 나온 해쉬값은
        1. 원본 메시지가 바뀌면 해쉬값이 바뀌어야 하고(Collision-Resistant)
        2. 예측하기 매우 어려워야 한다.(One-Wayness)

### Cryptographic Hash Function
- 대표적 해쉬 함수
    - MD5(128비트): 현재는 보안 관련 용도로 쓰는 것을 권장하지 않음
    - SHA: SHA-1, SHA-2, SHA-3
        - SHA-256, SHA-384, SHA-512는 모두 SHA-2
        - SHA-3도 있는데 기존의 SHA-2와는 전혀 다른 알고리즘임

    | 알고리즘 | 메시지 문자 크기 | 블록 크기 | 해시 결과 값 길이 | 해시 강도 |
    | --- | --- | --- | --- | --- |
    | SHA-1 | $<2^{64}$ | 512비트 | 160비트 | 0.625 |
    | SHA-256 | $<2^{64}$ | 512비트 | 256비트 | 1 |
    | SHA-384 | $<2^{128}$ | 1024비트 | 384비트 | 1.5 |
    | SHA-512 | $<2^{128}$ | 1024비트 | 512비트 | 2 |
- 암호화와 해슁을 헷갈리지 말기
    - 암호화: ‘평문 → 암호문 → 평문’ 이 가능해야 함
    - 해쉬: ‘평문→암호문’은 가능하나, ‘암호문→평문’ 은 불가능하다.

<br>

## Birthday Paradox
- $n$명이 함께 있을 때, 모두 다른 생일을 가질 확률은 생각보다 낮다.
- 하지만 $n$명 중에 '나'와 같은 생일자가 없을 확률은 생각보다 크다.

### Meaning
- The following two problems are quite different
- Find two message $M_1$ and $M_2$ such that $h(M_1)=h(M_2)$
- Given a message $M_1$, find another message $M_2$ such that $h(M_1 )=h(M_2)$
- 결론: original message을 hasing한 결과와 동일한 hasing값을 갖는 another message를 찾는 것은 매우 힘들다.
