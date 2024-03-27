---
title: Signatures 1
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

<br>

## RSA Signature

- Setting
    - $p$ and $q$ : large Primes,   $n=pq$,   $e$ : Encryption Key,   $d$ : Decryption Key
    - $ed=1 \mod {(p-1)(q-1)}$
    - $\varphi(n)=\varphi(pq)=(p-1)(q-1)$ (오일러 정리)
        - $pq-pq*\left(\frac 1p+\frac 1q\right)+1$
- Signature for Message $m$ (Only the Owner of Key)
    - $s(m) = m^d \mod n$
    - 메시지를 디크립트 → 나만 만들 수 있음
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

## Problems with RSA and ElGamal

- 현실적으로 RSA와 ElGamal을 사용하기에는 조금 느리다.
- Solution?
    - Classical Cryptosystem!

### Which Classical Systems are Alive
- Substitution - 단순 문자 치환
    - 카이사르 암호(Caesar cipher)
    - Change Letters by Rule, A→C, B→T, C→Q, …
    - Easily broken
- Exchange
    - 단일 문자 대응 암호(monoalphabetic cipher)
    - Change Location by Rule, ABCDEF->CAEDFB
    - Easily broken
        - 문자 e와 t가 많이 등장한다는 점, 특정 단어(in, the 등)이 쌍으로 자주 등장한다는 점, 메시지의 내용을 짐작..
        - 여러 정보를 이용하면 쉽게 깨질 수 있다.
- Substitution + Exchange 조합
    - 블록 암호(block cipher): 메시지를 k비트 블록으로 쪼개어, 각 블록을 암호화
    - Not broken yet → Feistel Cryptosystem and Variants
    - DES(Data Encryption Standard, 56bit), **AES(Advanced Encryption Standard, 256bit)**, IDEA, …
    - NIST에 따르면, 56비트 DES를 1초에 깰 수 있는 기계로 128비트 AES 키를 깨려면 149조 년이 걸림


<br>

## Typical Usage(전형적인 사용 패턴)

- 상황 : $A$가 $B$에게 메시지 $m$을 보내자
    - $B$가 공개한 key : $n_B, e_B$
    - $B$의 비밀키 : $d_B$

![](4.png)

Step1. $A$는 랜덤한 $k$를 만들어서 AES로 암호화 함

- $m \rightarrow AES(k)\rightarrow C$

Step2. $K$를 RSA로 암호화하는데, 이때 $B$의 공개키인 $n_B, e_B$를 사용한다.

- $K \rightarrow RSA(n_B, e_B) \rightarrow K'$

Step3. 이제 $A$가 $B$에게 $(K', C)$를 보낸다.

Step4. $B$는 개인키를 이용해 $K$를 얻은 후 $C$를 AES 복호화해서 $m$을 얻는다.

- RSA encrypts only the Key for AES
- AES encrypts the Message

## Signature
- RSA Signature은 오리지널 문서와 크기가 똑같다.
    - **길이가 너무 길다**
- Signature가 문서 사이즈에 의존하지 않고, 예측 못하는 적절한 고정된 길이를 사용하자.
    - **Hashing!**