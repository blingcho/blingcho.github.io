---
layout: post
title: "5-2 HW for Isolation"
categories:
  - Study
tags:
  - security
  - study
---

### Approaches for isolation

 - Process isolation
   - supervised by OS kernel as the reference monitor
 - VM or OS isolation
   - supervised by hypervisor as the reference monitor
 - Hardware-based TEE for isolation : 
   - crypto-hardware for RTM, RTS and RTRL: TCG TPM 
   - supervised by (HW-protected) privilieged SW : ARM Trustzone 
   - protected by hardware against privileged SW : intel SGX

### Process isolation
 - isolation between misbehaving process(red) and others(green) 
 - implement at two levels
 - inter-process isolation : 프로세스 사이에 isolation
 - in-process : 스레드, code segment 등 같은 address space에 있는 애들끼리 isolation

### [inter=]process isolation
 - Goal : OS에서 공격받은 다른 프로세스로부터 방어
 - virtual memory(virtualization) : mapping 되는걸 점검해서 isolation(paging을 통해)

### HW support for paging
 - MMU와 같은 HW 활용
 - permission check 
 - SW로 mapping하면 너무 느림

### MMU architecture
 - TLB(Translation Lookaside Buffer) virtual/physical pair를 가지고있다가 요구하는 address가 있으면 cache처럼 반환
 - miss일 경우 PTE(Page Table Entry)에서 첫번째 주소 가져와서 TLB를 업데이트하고 physical 주소에 가져온다

### Process isolation with MMU
 - 페이지 테이블은 OS or high privilieg SW에서 관리

### In-process isolation
 - Goal : prevent attack on a piece of a process from corrupting the entire process
 - Solution : address space를 나눈다

### Separate privilies
 - Monolithic design : 모든 오브젝트는 각각 하나의 정보에 접근
 - the priciple of least privilege

### Threads in separate address spaces
 - Arbiter, SMV : 스레드별로 그룹을 분리 (10% overhead)

### Splitting a process
 - 하나의 프로세스를 여러개의 프로세스로 쪼갬(inter-process 처럼)
 - split할때마다 오버헤드 증가
 - Privtrans : monitor를 만들고 필요할때마다 fork해서 slave(submodule)를 만들고 여기서 작업을 처리

### Software Fault Isoliation(SFI)
 - load, store, jump에 대하여 guard를 붙인다(instrument)
 - 같은 주소공간에서 동작하는 multiple code segments로 나눈다

### Adding guards
 - segment 안에서만 jump, load, store 가능하게
~~~
r12 <- [r11]
dr1 <- r11 & segment-mask
dr1 <- dr1 | dr2
r12 <- [dr1]
~~~

### Bypassing guard
~~~as
0xa0 : r10 <- 0xcc
0xa4 : jmp r10
   ...
0xc0 : dr1 <- r11 & segment-mask
0xc8 : dr1 <- dr1 | dr2
0xcc : r12 <- [dr1]
~~~

### Aligned jump
~~~as
0xa0 : r10 <- 0xcc
0xa4 : and r10, #0xf0
0xa8 : jmp r10

0xc0 : dr1 <- r11 & segment-mask
0xc8 : dr1 <- dr1 | dr2
0xcc : r12 <- [dr1]
~~~

### MMU for in-process isolation
 - SFI는 compile과정이 필요
 - code segment마다 PT permission을 다르게 할당
 - 해당 code segement를 사용할때 알맞는 page table을 loading해서 사용
 - 단점 : OS가 PT를 바꾸기 위해 intervention이 많음, page단위에 맞게 align해야해서 memory를 더 쓴다

### Isolation in tiny devices
 - MPU 사용
 - MPU(Memory Protection Unit)를 통해 8/16개로 영역을 나눔

### Process isolation with MPU
 - 실행되는 영역에 따라 MPU permission이 바뀜
