---
layout: post
title: "5-3 HW for Isolation"
categories:
  - Study
tags:
  - security
  - study
---

### VM/OS Isolation
 - VM Manager를 통해 VM을 isolation

### Nested paging
 - physical address -> machine address로 매핑
 - nested Page Table

### Shadow PT
 - PT 두개기 때문에 MMU가 두개 필요함
 - 초기에는 MMU한개로 support
 - VA -> PA -> MA를 VA -> MA로 single level mapping

### Terra [SOSP '03]
 - 어플리케이션은 몇몇 그룹으로 나누고 각자 OS 커널을 사용하게함
 - Hypervisor는 os kernel간에 isolation을 담당
 - hypervisor를 사용한 최초 연구 (Shadow PT 사용)

### Lares [S&P '08]
 - SPT를 통해 security VM(monitor application/techniques 탑재)을 isolation
 - guest OS에서 violation이 발생하면 securityVM으로 트램펄린해서 처리

### MMU extended for nested paging
 - SPT는 expensive(VM <-> hypervisor)
 - Extended PT(intel), ARM 2-stage(ARM)

### OSck [ASPLOS '11]
 - OS verifier를 통해 Guest VM들을 감시
 - Host Kernel에서 hypervisor기능을 가지고 있으면서 EPT를 통해 guest VM과 OS verifier를 isolation(같은 공간에 mapping되어도)

### Micro/thin-hypervisor
 - hypervisor를 통해 guest os를 올리고 여기서 memory관리 기능을 통해 security 획득
 - 큰 hypervisor 말고 security를 위한 작은 micro hypervisor

### InkTag [ASPLOS '13]
 - 하나의 OS에 대한 isolation
 - security critical application을 hypervisor가 보호
 - OS조차 HAP에 접근 못하도록 EPT를 trusted-EPT, untrusted-EPT를 만듬
 - HAP이 데이터를 전달할때 hypercall을 통해함

### Isolation without hypervisor
 - hypervisor를 쓰면 layer가 많아지니까 TLB miss rate이 증가

### Recall: in-process isolation
 - in-kernel에서도 process안에서 여러 파트로 나누는 개념을 적용해보겠다

### Privilege separation
 - SW entity가 접근할 수 있는 부분을 privilege로 구분한다
 - inter-/in-process isolation은 OS kernel에서 관리
 - VM/OS isolation은 hypervisor에서 관리
 - VM/OS isolation을 hypervisor없이 해보자

### In-kernel isolation
 - inner kernel은 높은 권한
 - outer kernel은 낮은 권한
 - process가 OS를 공격하더라도 outer kernel만 공격당하겠다

### Nested Kernel
 - in-kernel isolation technique
 - deploy에 문제가 있음
 - PT update instruction -> request to Nested kernel(PT management)

### Others
 - HILPS : TCR, TTBR를 잘 조절하면 OS kernel, hypervisor 모두 isolation 다 할수있다.
 - intra-level isolation
 - domain swithcing(inner <-> outer 구분)