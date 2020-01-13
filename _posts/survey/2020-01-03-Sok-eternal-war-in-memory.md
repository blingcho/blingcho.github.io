---
layout: post
title: "SoK: Eternal War in Memory"
categories:
  - Summary
tags:
  - security
  - survey
---

유형별 메모리 공격 기법과 보호 기법 소개 및 정리한 페이퍼. 


# Attack
---
* ## **Memory Corruption**
    
memory corruption은 보통 2단계로 구성된다.  

1. 포인터가 invalid하고 
2. 해당 포인터를 dereference

**Invalid pointer**

- out-of-bounds 된 포인터는 a spatial error

- free된 영역을 가르키는 포인터는 a temporal error

**A spatial error**
- 일반적인 indexing bug, buffer overflow
- null pointer
(allocation failure를 발생시켜서), 커널 단에서 null pointer의 dereference는 exploitable이라고 표현

**A temporal error**
- dangling pointer, free된 object를 가르키는 포인터
- 따라서 대부분 heap allocated object 대상
- global pointer가 local variable을 가르킬 때 dangling pointer 될 수 있음(함수가 종료될 때 스택프레임이 사라지게되어  가르키는 object가 미존재)
  
---
* ## **Code corruption attack**

**Code integrity**

- Memory corruption을 이용해서 code를 overwrite 가능
- 이에 대한 integrity를 보장하기위해 현대 프로세서는 read-only memory 지원됨.
- 그러나 JIT 상황에서는 어려움(e.g. browser, js, flash, etc..)

---
* ## **Control-flow hijack attck**

**Code pointer integrity**
- 마찬가지로 memory corruption을 통해 code pointer 부분을 수정 -> control flow 조작 가능
- 공격자가 jump할 address를 특정해야하기 때문에 이를 이용해서 랜덤화하는 방식으로 integrity를 지키려고 함

**Write XOR Execute**
- Control-flow hijack을 통해 stack, heap 영역으로 점프해서 코드를 실행시키는 것을 막기위함
- Code integrity랑 같이 생각해서 한 영역에서 write랑 execute를 못하게 page permission을 설정(cpu가 지원)
- 마찬가지로 JIT 상황에서 어려움

**Instruction Set Randomization**
- code를 encrypting하여 inject나 corrupt 예방
- 그러나 cpu의 page permission을 이용한 방식보다 느리기 때문에 더 이상 다루지 않음

**Code reuse attack bypassing W^E(ROP)**
- Control-flow를 hijack하여도 임의의 코드를 실행할 수 없기 때문에(W^E) 가젯들을 모아서 실행
- 딱히 막을 방법이 없어서 최근 연구들은 code gadgets을 [컴파일러](http://repository.bilkent.edu.tr/bitstream/handle/11693/28479/bilkent-research-paper.pdf?sequence=1), [binary rewriting](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=6234439) 등을 통해 없애는 연구를 진행
- 그럼에도 불구하고 어려워질뿐 막아지는건 아님
- 상위 정책 관련 논문 [SFI](http://users.ece.cmu.edu/~adrian/630-f03/readings/sfi.pdf), [XFI](https://www.usenix.org/legacy/event/osdi06/tech/full_papers/erlingsson/erlingsson.pdf), [Native Client](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=5207638)


---
* ## **Data-only attack**

**Data-only attack**
- 위에 언급한 code나 code pointer를 수정하지 않고 if-statement에 쓰이는 변수를 수정해서 control flow를 조작
- Code integrity, Code pointer integrity 뿐만 아니라 이런 변수에 관한 integrity까지 합쳐 data integrity라고 표현
- Address Space Randomization을 확장하여 Data Space Randomization(모든 데이터 랜덤화)

---
* ## **Information leak**
  
**Information leak**
