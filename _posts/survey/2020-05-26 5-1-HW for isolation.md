---
layout: post
title: "5-1 HW for Isolation"
categories:
  - Study
tags:
  - security
  - study
---

### Attack surface

 - 넓다 좁다처럼 정성적으로만 표현 가능, 10-20 정량적 불가능
 - 취약성 통로의 합
 - 공격자가 이용하는 버그들 ~ 이런 느낌

### Attack vector
 - 취약점에서 공격자가 들어올 수 있는 수단, 툴, 방법(virus, e-mail attachments, chat rooms)


### How to reduce attack surface
 - 전체 코드의 양, entry point, interface를 줄여야한다.
 - Least privilege : each user/program must operate using the fewest privilege
 - Least common mechanism : minimize the amount and use of shared mechanisms

### Isolation reduces attack surface
  - limit privilege to access
  - restrict reckless, open access

### Isoation
 - Access control
 - 민감한 정보에 대한 모든 외부 접근 차단 
 - 보안 정책에 맞춰 선별적으로 control access

### Normal system model
 - Less trustworthy, Less accountable이 많다
 - More trustworthy, More accountable이 적다
 - 이는 곧 isolation 안되어있으면 전자로부터 후자 공격 가능

### Red/Green Model for Isolation
 - Red가 전자, Green이 후자 분리시켜 논다

### TEE(Trusted Execution Environment)
 - 외부로부터 조작가능하지 않은 코드/데이터를 내부에 올려서 실행
 - 모든 entitied로 부터 코드/데이터를 방어
 - isolated execution -> access control(based on security policies)이 제공되는 독립적인 공간에서 수행
 - **remote attestation을 보장해야함**

### TEE - Security Policies
 - Data separation : partition 나누기
 - Sanitization : free했을때 정보없이 반남
 - Data flow control : data가 올바르게 왔다갔다.
 - Fault isolation : 특정한 sector 중에 fault가 나면 해당 sector안에서만 해결

### TEE - Desired primitives
 - Secure storage
 - Isolated execution
 - Platform integrity
 - Device identification and authentication

### Root of Trust
 - Trust anchor
 - RTM : RoT for measuring trustiness -> DRTM/SRTM
 - RTR : RoT for reporting the measurement
 - RTS : RoT for storage

### TEE with RoT(ARM TrustZone)
 - ROM에 RoT 박아놈
 - 그 위에 Monitor w/ TEE entry
 - 그 위에 secure world를 올림
 - 이걸 chain of trust

### TCB(Trusted Computing Base)
 - security policy를 enforce하는 entity
 - OS kernel, Secure OS, Secure monitor, Hypervisor, SGX sw/hw for enclave application

### SEE(Secure Execution Environment)
 - SEE > TEE
 - RoT를 꼭 필요로하지 않음. 꼭 외부에 자신이 안전함을 알릴 필요 X (TEE는 RTM 씀)
 - Confidentiality, Integrity, Authenticity

### Reference Monitor
 - TCB가 monitoring함
 - privilege가 높아야하고, 따라서 최대한 작아야한다

### TEE system architecture for isolation
 - SW-based TEEs : OS kernel, Hypervisor, Nested Kernel
 - HW-based TEEs : Trusted Platform Module, ARM TrustZone, Intel SGX
