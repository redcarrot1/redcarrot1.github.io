---
title: "Digital Envelope"
date: 2024-12-23 10:00:00 +09:00
categories: [암호학]
tags:
  [
     암호학, cryptography
  ]
math: true
img_path: /assets/img/cryptography/
---

## Hybrid crypto system
- 상황 : $A$가 $B$에게 메시지 $m$을 보내자
  - $B$가 공개한 key : $n_B, e_B$
  - $B$의 비밀키 : $d_B$

![](4.png)

Step1. $A$는 랜덤한 $k$를 만들어서 AES로 암호화 함 (대칭키)

- $m \rightarrow AES(k)\rightarrow C$
- 이때 $k$를 '세션키'라고 부르기도 함

Step2. $K$를 RSA로 암호화하는데, 이때 $B$의 공개키인 $n_B, e_B$를 사용한다. (비대칭키)

- $K \rightarrow RSA(n_B, e_B) \rightarrow K'$

Step3. 이제 $A$가 $B$에게 $(K', C)$를 보낸다.

Step4. $B$는 개인키를 이용해 $K$를 얻은 후, $C$를 AES 복호화해서 $m$을 얻는다.

Step5. 이후부터는 메시지를 대칭키($k$)를 이용해 암호화하여 전송

- **RSA encrypts only the Key for AES**
  - key exchange
  - [RSA](https://redcarrot1.github.io/posts/RSA/), [Diffie-Hellman](https://redcarrot1.github.io/posts/Diffie-Hellman,ElGamal/#diffie-hellman-key-exchange), [ElGamal](https://redcarrot1.github.io/posts/Diffie-Hellman,ElGamal/#elgamal), ...
- **AES encrypts the Message**
  - [Symmetric key Cryptography](https://redcarrot1.github.io/posts/symmetric_key_cryptography/)
  - DES, 3DES, AES, ...


<br>

## Digital Envelope

- RSA encrypts only the Key for AES
- AES encrypts the Message
- SHA hashes the Message
- RSA makes Signature with Hash Value
- *RSA, AES, SHA, RSA Signature는 메커니즘의 한 구현 방법일 뿐이다.
  
![](5.png)