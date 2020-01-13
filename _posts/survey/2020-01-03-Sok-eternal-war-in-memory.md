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

___
# Attack
<br>

## **Memory Corruption**
    
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
  
## **Code Corruption Attack**

**Code integrity**

- Memory corruption을 이용해서 code를 overwrite 가능
- 이에 대한 integrity를 보장하기위해 현대 프로세서는 read-only memory 지원됨.
- 그러나 JIT 상황에서는 어려움(e.g. browser, js, flash, etc..)

## **Control-flow Hijack Attack**
<br>

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


## **Data-only Attack**

**Data-only attack**
- 위에 언급한 code나 code pointer를 수정하지 않고 if-statement에 쓰이는 변수를 수정해서 control flow를 조작
- Code integrity, Code pointer integrity 뿐만 아니라 이런 변수에 관한 integrity까지 합쳐 data integrity라고 표현
- Address Space Randomization을 확장하여 Data Space Randomization(모든 데이터 랜덤화)

## **Information Leak**
  
**Information leak**
- information leak을 통해 randomization 관련 솔루션을 우회 가능
- 약화시킬 수 있는건 full-data randomization

___
# Currently Used Protections
<br>

## **Stack smashing protection**
- saved return address 아래에 canary 삽입 후 return 시 체크
- Control-flow integrity
- SafeSEH, SEHOP, etc
  
## **DEP W^X**
- Execution과 Write 중 하나만 가능
- code reuse attack에는 취약(e.g. ROP)

## **ASLR**
- 힙, 스택, 공유 라이브러리, main code segment를 가상주소공간에 mapping

___
# Approaches and evaluation criteria
- 확률론적인 방법과 결정론적인 방법으로 나뉨
- 확률론적인 방법은 ISR, ASLR, DSR 등등
- 결정론적인 방법은 그 외 대부분, reference monitor
- dynamic 또는 static하게 instrumentation함
- dynamic은 unsafe한 바이너리를 대상으로 할 수 있고 느린 대신, static은 소스코드가 필요하고 빠르다
- 간단한 reference monitor 종류는 낮은 오버헤드로 구현가능([taint checking](http://www.icode-project.eu/media/page-media/7/minemu-raid11.pdf), [ROP detector](https://www.researchgate.net/profile/Marcel_Winandy/publication/221609042_ROPdefender_A_detection_tool_to_defend_against_return-oriented_programming_attacks/links/0fcfd50d09cae46eb2000000/ROPdefender-A-detection-tool-to-defend-against-return-oriented-programming-attacks.pdf))


## **Protection**
1. **Enforce policy** : 사용하는 정책이 효과적인 지?
2. **False negatives** : 미탐지는 없는지?
3. **False positives** : 오탐지는 없는지?

## **Cost**
1. **Performance overhead** : CPU-bound/IO-bound 특히 CPU-bound에 대한 performance overhead가 적어야함 통상 10% 정도가 deploy됨
2. **Memory overhead** : inline monitor 같은 경우가 memory overhead 발생, performance overhead보다는 이쪽 오버헤드가 나음

## **Compatibiliry**
1. **Source compatibility** : 수작업으로 소스코드를 수정하지않아도 적용 가능
2. **Binary compatibility** : unmodified된 binary module이 있어도 상관없음(unmodified binary에 대한 satety를 제공하지 않더라도)
3. **Modularity support** : 각각의 모듈로 다뤄야한다(이해 잘안감). 
   
___
# Probabilistic Methods
<br>

## **Address Space Randomization**
- 계속 나왔던 그 ASLR
- 자료마다 randomly arrange 하는 영역이 다름. 본 논문에서는 code and data 영역이라고 표현하는데, 여기서 data space randomization과 차이점을 모르겠음([ASLR](https://pax.grsecurity.net/docs/aslr.txt), [DSR](http://seclab.cs.sunysb.edu/seclab/pubs/dsr.pdf))
- 32bit 머신에서는 randomization의 entropy가 낮아서 효과적이지 않음(brute force, de-randomization 공격에 취약)
- 모두 randomized하더라도 information leak으로 우회할 수도 있음
- DEP W^X가 있으니까 ASLR의 주된 포인트는 code
- ASLR이랑 DSR 중간쯤으로 [PointGuard](https://www.usenix.org/legacy/event/sec03/tech/full_papers/cowan/cowan_html/)이 있고, 해당 논문은 하나의 키만으로 xor encryption을 해서 쉽게 복구가능([링크](https://lirias.kuleuven.be/retrieve/60100))

## **Data space randomization**
- ASR처럼 location을 randomization하는게 아니라 representation을 랜덤화
- 같은 "points-to" sets에 포함되는 variables는 같은 키 사용
- 평균적으로 15%의 오버헤드
- binary compatibility X
- 전체적으로 points-to analysis가 필요하기 때문에 modularity X. 이를 해결하기 위해 부분적인 points-to analysis를 제안

___
# Memory Safety
- 
<br>

## **Data space randomization**