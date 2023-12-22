---
title: Number Theory 1
date: 2023-12-14 10:00:00 +09:00
categories: [암호학]
tags:
  [
    암호학, cryptography
  ]
math: true
---

## Divisibility, Divisor, Multiple, Remainder
본격적으로 암호학 정수론을 시작하기 전에 Notation을 소개한다. 또한 앞으로의 포스팅에서는 수가 음수가 되거나 0이 되는 경우는 대부분 제외한다.

아래는 모두 같은 것을 의미한다.
- $b=ca$
- $a\|b$
- $a$ divides $b$
- $a$ is a divisor(약수) of $b$
- $b$ is a multiple(배수) of $a$

Prime $p$ : only $1, -1, -p, p$ divides $p$

### Some Properties
- 약수는 다음과 같은 특징을 가진다. 결과보다는 증명 과정을 깊게 보면 좋을 것 같다.
- $a\|b \iff b=ac $ 인 정의를 사용해서 증명하자
    - 위 수식을 만족하는 $c$를 찾으면 된다.

- $1\|a, -1\|a, a\|0$
    - $1\|a \rightarrow a=c*1 \rightarrow c=a$
    - $-1\|a \rightarrow a=c*(-1) \rightarrow c=-a$
    - $a\|0 \rightarrow 0=c*a \rightarrow c=0$
- $a\|a, a\|-a, -a\|a, -a\|-a$
    - 차례로 $c=1, -1, -1, 1$
- $a\|b$ and $a\|c$, then $a\|(bx+cy)$
    - $a\|b \rightarrow b=sa$인 $s$가 존재
    - $a\|c \rightarrow c=ta$인 $t$가 존재
    - $bx+cy=sax+tac=(sx+tc)*a \rightarrow$ $a$로 나누어 떨어짐
- $a\|b$ and $b\|c$, then $a\|c$
    - $a\|b \rightarrow b=sa$인 $s$가 존재
    - $b\|c \rightarrow c=tb$인 $t$가 존재
    - $c=(ts)*a \rightarrow$ $a$로 나누어 떨어짐

## Division Theorem
> 나누기의 결과는 항상 한개이다.

For integers $a(\ne0), b$, there exist unique integers $q$ and $r$ such that, $b=qa+r, 0\le r<\|a\|$

**Proof by contradiction**
- 나누기의 결과가 1개 이상인 만족하는 조합이 존재한다고 가정하자. $(q_1, r_1, q_2, r_2)$
  - $b=q_1a+r_1, b=q_2a+r_2$
- 만약 $r_1=r_2$이면 자연스럽게 $q_1=q_2$ 이다 : **모순**
- 만약 $r_1\ne r_2 (r_1<r_2)$ 이면, $0<r_2-r_1<a$ 이고,
    - $0=(q_2a+r_2)-(q_1a+r_1)$ 이다.
    - 왼쪽 $0$은 $a$로 나누어 떨어지지만, 오른쪽은 $a$로 나누어 떨어지지 않는다. : **모순**
- 따라서 정수 나누기는 결과가 단 1개이다.


## Greatest Common Divisor (GCD)
**Definition**
- for integers $𝑎$ and $𝑏$ unique integer $𝑑$ such that
  - $𝑑\|𝑎$ and $𝑑\|𝑏$ : $d$는 $a, b$의 공약수이다.
  - for all possible $𝑐$, $𝑐\|𝑎$ and $𝑐\|𝑏$ implies $𝑐\|𝑑$ : $c$가 $a, b$의 공약수이면 $d$로 나누어 떨어져야 한다.

$a, b$의 최대공약수를 앞으로 $gcd(a, b)$로 표현한다.
- $a, b$는 서로소(Relatively Prime) $\iff gcd(a,b)=1$
- $a, b$의 공약수는 최대공약수의 약수들로 이루어져 있다.
    - $a$의 약수 집합 $A$, $b$의 약수 집합 $B$가 존재한다고 하자.
    - $a, b$의 공약수 집합 : $A \cap B$
    - $gcd(a, b)$ = $max(A\cap B)$


## How to Calculate GCD?
- Euclid Algorithm for $a$ and $b$ ($0<a<b$)
    - $b=q_1a+r_1$
    - $a=q_2r_1+r_2$
    - $r_1=q_3r_2+r_3$
    - $\cdots$
    - $r_{k-2}=q_kr_{k-1}+r_k$
    - $r_{k-1}=q_{k+1}r_k +0$ $(r_{k+1}=0)$
    - 결론: $gcd(r_{k-1}, r_k)=r_k$

### Correctness of Euclid
증명할 명제
> If $b=q_1a+r_1$, then $gcd(b, a)=gcd(a,r_1)$

1. '$a, b$의 공약수 집합  $\subseteq a, r_1$의 공약수 집합'임을 증명
- $a, b$의 공약수 집합의 어떤 원소를 $c_1$라 하자
- $b=c_1*k_1, a=c_1*k_2$
- $b=q_1a+r_1$이므로, $c_1k_1 = q_1c_1k_2+r_1$
- $c_1(k_1-q_1k_2)=r_1$ 이다. 따라서 $r_1$은 약수로 $c_1$를 가지고 있다.

[반대 방향 증명]
2. '$a,r_1$의 공약수 집합  $\subseteq a, b$의 공약수 집합'
- $a, r_1$의 공약수 집합의 어떤 원소를 $c_2$라 하자
- $a=c_2k_3, r_1=c_2k_4$
- $b=q_1a+r_1$이므로, $b = q_1c_2k_3+c_2k_4$
- $b=c_2(q_1k_3+k_4)$ 이다. 따라서 $b$는 약수로 $c_2$를 가지고 있다.