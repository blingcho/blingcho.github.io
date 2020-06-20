---
layout: post
title: "5-4 HW for Isolation"
categories:
  - Study
tags:
  - security
  - study
---

### Trusted Platform Module [TPM]
 - TCG(Trusted Computing Group) 비영리기관이 HW trusted computing standart 제안
 - 암복호화 기능, 키 관리 기능, signing, hashing

### Samsung TPM  
 - 칩에 ARM과 별도인 자체적인 trust 시스템 빌드
 - HW RoT 칩 제작

### TPM Solution
 - secure storage
 - integrity measures
 - attestation identity key
 - Program Code
 - Exec Engine
 - Random Number Generator
 - Hash engine
 - Key generator
 - RSA Engine

### Security functions of TPM
 - 안전한 storage를 TPM에 넣고 이를 신뢰
 - TPM -> RSA 사용
 - Root of Trust로 동작
 - sealed storage
 - authentication : who are you
 - remote attestation

### TPM keys
 - EK (endorsement key)
 - SRK (Storage root key)
 - Storage keys
 - AIKs (attestation identity key)

### Loading/unloading keys & data
 - TRM안에는 키를 많이 저장 어려워서 key hierachy

### [Attestation] Identity keys
 - AIK를 쓰고싶으면 TPM를 통해 

### Integrity measure
 - PCR에 hash값을 저장
 - 주기적으로 hash값 update

### What to measure
 - 부팅될때 OS/kernel 을 점검
 - chain of transitive trust
 - BIOS bootblock는 ROM에 저장 그 다음꺼의 hash값을 저장

### Remote attestation
 - 