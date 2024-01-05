---
title: Discrete Log Problem
date: 2024-01-05 10:00:00 +09:00
categories: [암호학]
tags:
  [
    암호학, cryptography
  ]
math: true
img_path: /assets/img/cryptography/
---

## Discrete Log Problem
### Generator
> if $Z_n^{\*}=\lbrace g, g^2, \dots, g^{\varphi(n)}(=1)\rbrace$, then $g$ is a Generator of $Z_n^{\*}$

예를 들어, $n=7$이라면 $Z_n^{\*}=\lbrace 1, 2, 3, 4, 5, 6\rbrace$이다. $3^1=3, 3^2=2, 3^3=6, 3^4=4, 3^5=5, 3^6=1 \mod 7$ 이므로 $3$은 $Z_7^{\*}$의 Generator이다.<br>
추가) $Z_n^{\*}$ contains a Generator $\iff n=1, 2^2, p^k, 2p^k$ where $p$ is an odd Prime (증명생략)



### Discrete Log Problem
> Discrete Log Problem: Given $n, g, x$, find $k$ such that $g^k=x \mod n$

한쪽은 계산하기 어렵고, 반대 방향은 계산하기 쉬운 함수를 one-way function이라고 한다.<br>
DL Problem에서 $n, g, k$가 주어지면 $x$를 찾는 건 쉽지만, $n, g, x$가 주어질 때 $k$를 찾는건 어렵다.<br>
DL Problem은 $NP \cap co-NP$ 문제로 알려져있다.


## Diffie-Hellman Key Exchange

- 대칭키시스템에서 사용할 key를 생성하는 과정
![](2.png)

- Setting
    - public : $p, g$ (shared by all)
    - Alice’s Public Info: $g^a \mod p$
    - Alice’s Secret Info: $a$
    - Bob’s Public Info: $g^b \mod p$
    - Bob’s Secret Info: $b$
    - $g^a$에서 $a$를 알아내는건 DL 문제임
- Shared Key Generation
    - Alice computes : $(g^b)^a = g^{ab} \mod p$
    - Bob computes : $(g^a)^b = g^{ab} \mod p$
    - $g^a, g^b$를 알고있는 제 3자인 C가 $g^{ab}$를 알아낼 수 있을까? 글쎄… 쉽게 알아내지는 못하는 것 같다.

<br>

## ElGamal

![](3.png)

### Setting
- public : $p, g$ (shared by all)
- Alice’s Public Info: $g^a \mod p$
- Alice’s Secret Info: $a$
- Bob’s Public Info: $g^b \mod p$
- Bob’s Secret Info: $b$

### ElGamal Send Message $𝑚$

- Bob (Sender) computes:
    - Random $k$
    - Send $(g^k \mod p, mg^{ak} \mod p)$ to Alice
- Alice (Receiver) computes:
    - $(g^k)^a=g^{ak} \mod p$
        - $A$는 $a$를 가지고 있으므로 구할 수 있다.
    - $mg^{ak}(g^{ak})^{-1}=m \mod p$
        - $g^{ak}$를 알고있으므로 곱셈의 역원(유클리드)을 구할 수 있음
        - 따라서 원래 메시지 $m$을 구할 수 있다.