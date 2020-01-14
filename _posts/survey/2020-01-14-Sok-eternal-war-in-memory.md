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
- Address Space Randomization을 확장하여 Data Space Randomization

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
<br>

## **Spatial safety with pointer bounds**
- spatial memory safety를 위해서는 pointer bounds를 추적해야함
- [Softbound](https://repository.upenn.edu/cgi/viewcontent.cgi?article=1941&context=cis_reports)는 metadata를 pointer와 떨어뜨려 compatibility를 해결(이전에는 소스코드 annotation을 통해 pointer structure가 metadata를 가지게 하는 방법이 제안되었음/큰 소스코드에 부적합)
- 이 방법은 모든 모듈이 적용될 경우 FP, FN 모두 제로
- 그러나 평균적으로 오버헤드가 67%로 매우 느림
- 그리고 unprotected library와는 제한된 compatibility 제공(metadata update가 안되서 FP 발생)

## **Spatial safety with object bounds**
- pointer에 metadata를 연결하는게 아닌 object에 연관지음
- 할당된 메모리의 영역을 아는 것 만으로는 불충분 -> 포인터가 다른 오브젝트를 가르키고있어도 모름
- 역참조되지 않으면 out-of-bound 가능
- 해당 방법으로 접근한 논문 [CRED](http://www.cs.cmu.edu/afs/cs.cmu.edu/Web/People/oor/papers/cred.pdf)
- 구조체 내의 포인터에 대한 탐지 불가능
- CRED 오버헤드는 2x, a static points-to analysis로 1.2x까지 낮춤([automatic pool allocation](http://www.cs.cmu.edu/afs/cs/academic/class/15745-s06/web/handouts/lattner-pldi05.pdf))
- 현재 가장 빠른 방식은 [Baggy Bounds Checking](https://www.usenix.org/legacy/event/sec09/tech/full_papers/sec09_memory.pdf), 60%의 오버헤드

## **Temporal safety**
- spatial safety 외에 uaf, double free 같은 취약점도 존재.
- 이러한 취약점은 기존의 bound checking으로 탐지 불가능
1. **special allocators** : 한번 할당한 가상메모리를 절대 다시 사용하지 않음으로 해결. 낭비가 심하고 dangling pointer 같은 문제는 해결하지 못함
2. **object based approaches** : location을 마킹해서 하는 방식. 그러나 valgrind는 dynamic translator라 느리고, AddressSanitizer는 73% 오버헤드 발생. re-allocation된 경우도 탐지 못함.
3. **pointer based approaches** : pointer에 bounds information 뿐만 아니라 allocation information도 남기면 full-memory safety 가능. 그러나 pointer가 가르키는 object에 대한 bit만 추가해주면 found, update하기 어렵기 때문에 object validity bit는 global dictionary에 저장시켜놓고 이를 사용([CETS](https://repository.upenn.edu/cgi/viewcontent.cgi?article=1744&context=cis_papers)). softbound와 함께 사용하면 116% 오버헤드를 가지고 마찬가지로 binary compatibility 이슈가 있음.

___
# Generic Attack Defenses
<br>

## **Data integrity**
1. **safe objects** : static pointer analysis를 통해 unsafe한 오브젝트를 찾음(계산되어서 참조하는 값). unsafe object(unsafe pointer의 points-to sets)를 부를때 각 바이트를 마크하고 instrument함([이해안됨](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.65.1702&rep=rep1&type=pdf)/unsafe or not만 구분해서 마크한다는 의미로 추정됨). 오버헤드는 50%~100%정도이다.
2. **points-to sets** : points-to set마다 각자의 마크(식별ID/여러집합/같은 집합 안에서는 취약할 수 있음)를 shadow memory에 저장([WIT](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=4531158)). 특정 포인터는 point analysis를 통해 dereference 할 수 있는 points-to set '만' dereference 가능. 오버헤드는 525%정도이고 binary compatible을 만족하지 못한다.<br> safe object에서 언급된 방식에서 해결못한 read를 통한 control hijack을 해결하기 위해 indirect call과 관련된 points-to set도 생성.

## **Data-flow integrity**
- [Data-flow integrity](https://static.usenix.org/event/osdi06/tech/full_papers/castro/castro.pdf)
- 마지막 write instruction이 올바른지 read 시 체크
- write instruction마다 ID를 가지고 있고 정적으로 허용될 수 있는 ID를 미리 계산
- write를 체크하지 않기 때문에 metadata를 손상시킬 수 있음(의도적으로 오류를 발생시킬 수 있다?)
- 오버헤드는 50~100% 정도

___
# Control-Flow Hijack Defenses
<br>


## **Code pointer integrity**
- GOT, VT에 있는 포인터는 read-only로 보호할 수 있지만, 그 외에는 writable 해야됨(SRA, function pointer 등등)
- code pointer의 integrity를 보장하여도 control-flow hijack 가능
- e.g. UAF, dangling pointer 등등

## **Control-flow integrity**
1. **Dynamic return integrity** : SSP(Canary value), Shadow stack과 같이 saved return address에 대한 integrity. [shadow stack](https://static.usenix.org/event/osdi06/tech/full_papers/castro/castro.pdf) 같은 경우 saved return address를 shadow stack에 push하고 retunr시 program stack과 비교. <br>두 stack을 overwriting하면 취약해서 shadow stack을 보호하기 위해 guard pages, switching write permission 사용한 접근법도 있음([RAD](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=918971)). SSP의 경우 오버헤드가 1%보다 작고, shadow stack의 경우 unprotected의 오버헤드는 5%, 그러나 protected는 10x배 느려짐.
2. **Static control-flow graph integrity** : static하게 미리 control-flow graph를 만듬. WIT와 다르게 ID를 포인터 옆에 저장. jumping하기 전에 체크. runtime에서는 return address가 하나지만 이 방법에서는 여러개가 가능함. 
