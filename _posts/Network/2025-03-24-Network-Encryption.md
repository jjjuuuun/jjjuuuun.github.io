---
layout: post
title: "🌐 [Network] Encryption 🌐"
author: kjy
date: 2025-03-25 08:00:02 +09:00
categories: [Computer Science, Network]
tags: [Computer Science, Network]
comments: true
toc: true
math: true
---

## 🌐 암호화(Encryption) & 복호화(Decryption)

암호화(Encryption)란 원문 데이터를 알아볼 수 없는 형태로 변경하는 것을 말하고 복호화(Decryption)란 암호화된 데이터를 원문 데이터로 되돌리는 과정을 말합니다.

암호화의 종류에는 다음과 같은 것들이 있습니다.

|      | 대칭 키 암호화(Symmetric Key Cryptography) |             비대칭 키 암호화(Asymmetric Key Cryptography)             |
| :--: | :----------------------------------------: | :-------------------------------------------------------------------: |
| 설명 |    암호화와 복호화에 동일한 Key를 사용     | 암호화를 위한 Key(Public Key)와 복호화를 위한 Key(Private Key)가 다름 |
| 장점 | 적은 부하로 암호화 및 복호화를 빠르게 수행 |                       Key를 안전하게 공유 가능                        |
| 단점 |           Key가 유출 우려가 있음           |                           시간이 오래 걸림                            |

## 🌐 인증서와 디지털 서명

Network에서 사용되는 인증서(Certificate)는 일반적으로 공개 키 인증서(Public Key Certificate)를 의미합니다.

공개 키 인증서란 Public Key의 유효성을 입증하기 위한 전자 문서입니다.

Client와 Server가 공개 키 암호화 방식으로 통신할 때 Server로부터 전달받은 Public Key가 신뢰할 수 있는 Key인지 Client가 확신하기 위해 Server는 Public Key와 함께 조작되지는 않았는지, 유효기간은 언제까지인지 등의 정보를 포함하는 인증서를 전송합니다.

이러한 인증서는 인증 기관(CA; Certification Authority)인 제 3의 기관에서 발급 받습니다. 인증서에는 CA가 인증했다는 표시의 서명값(Signature)이 있습니다. Client는 Signature를 통해 인증서를 검증할 수 있습니다.

> 💡 서명값(Signature)  
> 📢 인증서 내용에 대한 해시 값을 CA의 Private Key로 암호화 하는 방식

공개 키 인증서로 Public Key의 유효성을 입증하는 과정(Digital Signature)을 살펴보겠습니다.

| 1️⃣ 인증서 내용에 대한 해시 값을 CA의 Private Key로 암호화(`Signature`) |
| 2️⃣ CA는 `Signature`와 `인증서`를 함께 Client에게 전달 |
| 3️⃣ Client는 `Signature`와 `인증서`를 분리 |
| 4️⃣ CA가 공개한 Public Key로 `Signature`를 복호화하여 인증서 내용에 대한 해시 값 획득 |
| 5️⃣ `인증서`에 대한 해시 값을 직접 구한 뒤 4번의 해시 값과 비교 |
| 6️⃣ 해시 값이 일치한다면 전달받은 `인증서`는 확실히 CA의 Private Key로 만들어졌다고 보장할 수 있으므로 Public Key의 유효성을 인증 |
