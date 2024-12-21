---
title: Number Theory 2
date: 2023-12-22 10:00:00 +09:00
categories: [암호학]
tags:
  [
    암호학, cryptography
  ]
math: true
---

## Extended Euclid
**Euclid Algorithm**
- $b=q_1a+r_1$
- $a=q_2r_1+r_2$
- $r_1=q_3r_2+r_3$
- $\cdots$
- $r_{k-3}=q_{k-1}r_{k-2}+r_{k-1}$
- $r_{k-2}=q_kr_{k-1}+r_k$

<br>
위의 유클리드 알고리즘에서 식을 유도하고자 한다. 아래쪽 식 2개만 가져와보자.<br>

- $r_{k-3}=q_{k-1}r_{k-2}+r_{k-1}$
- $r_{k-2}=q_kr_{k-1}+r_k$

위 식을 정리하면 다음과 같은 결과가 나온다.<br>

$r_k=r_{k-2}-q_kr_{k-1}=r_{k-2}(1-q_{k-1})-q_kr_{k-3}$

$r_k$가 $x\times r_{k-2}+y\times r_{k-3}$ 꼴로 표현됨을 볼 수 있다. <br>
지금은 식 2개만 가지고 정리했지만, 이와 같이 계속 진행하면 $r_k=x\times a+y\times b$ 로 나타낼 수 있다. ($r_k$는 최대공약수)

**결론: $x, y$가 정수일 때 $x\times a+y\times b$가 가능한 값은 gcd(a, b)의 배수이며, 오직 이 값들만 가능하다. 또한 식을 만족하는 $x, y$를 찾는 것이 가능하다.** ([베주 항등식](https://ko.wikipedia.org/wiki/%EB%B2%A0%EC%A3%BC_%ED%95%AD%EB%93%B1%EC%8B%9D))

<br>

## 곱의 역원
$a \times a^{-1}=1 \mod n$ 일 때, $a^{-1}$를 $a$의 곱의 역원이라고 한다.<br>
$a \times a^{-1}=\alpha n+1$로도 표현할 수 있는데, 식을 정리하면 $a \times a^{-1}-\alpha n=1$ 이다. <br>
$x\times a+y\times n=1$ 꼴이므로 반드시 $\gcd(a, n)=1$ 이어야 한다. 그래야 곱의 역원이 존재한다.

<br>

## Some Theorems
### Theorem 1
> $\gcd(a, b)=1 \iff$ some $x$ and $y$, $ax+by=1$
  
- 오른쪽 방향 증명
    - Extended Euclid(위)에서 증명했다.
- 왼쪽 방향 증명
    - $\gcd(a, b)=d$이면 $d\|ax+by$
    - $ax+by=1$이므로 $d\|1$ 이다. 이걸 만족하는 $d$는 $1$ 뿐이다.

### Theorem 2
> If $a\|bc$ and $\gcd(a, b)=1$ then $a\|c$
  
- $a\|bc \rightarrow bc=ka$
- $\gcd(a, b)=1 \rightarrow ax+by=1 \rightarrow acx+bcy=c\rightarrow acx+kay=c$
- $a\|c \rightarrow c=k^\prime a$
- $(cx+ky)*a=c=k^\prime a$이므로 만족
- 상식 선에서 생각해보면, $a, b$가 서로소인데 $a$는 $bc$의 배수라는 것은 $c$가 $a$의 배수라는 의미이다.

### Theorem 3
> If $\gcd(a, b)=d$ then $a=a_0d, b=b_0d, \gcd(a_0, b_0)=1$

- $ax+by=d$에서 $a_0dx+b_0dy=d \rightarrow a_0x+b_0y=1$
- $a_0x+b_0y=1 \iff \gcd(a_0, b_0)=1$

### Theorem 4
> $\gcd(a, b)=1$ and $\gcd(a, c)=1$ $\iff$ $\gcd(a, bc)=1$

- 오른쪽 방향 증명
  - $ax+by=1, as+ct=1$
  - 두 식을 양변 곱 → $(ax+by)(as+ct)=1$
  - $a^2xs+acxt+absy+bcyt=1$
  - $(axs+cxt+bsy)\times a+(yt)\times bc=1$
- 왼쪽 방향 증명은 $ax+bcy=1$로 증명(생략)
