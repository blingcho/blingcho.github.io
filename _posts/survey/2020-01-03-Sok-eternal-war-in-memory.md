---
layout: post
title: "SoK: Eternal War in Memory"
categories:
  - Summary
tags:
  - security
  - survey
---

# ***SoK: Eternal War in Memory***

유형별 메모리 공격 기법과 보호 기법 소개 및 정리한 페이퍼. 


# Attack
---
- ## **Memory Corruption**
    
memory corruption은 보통 2단계로 구성된다.  

1. 포인터가 invalid하고 
2. 해당 포인터를 dereference.

**Invalid pointer**

- out-of-bounds 된 포인터는 a spatial error. 

- free된 영역을 가르키는 포인터는 a temporal error.

**A spatial error**
- 일반적인 indexing bug, buffer overflow
- null pointer
(allocation failure를 발생시켜서), 커널 단에서 null pointer의 dereference는 exploitable이라고 표현

**A temporal error**
- dangling pointer, free된 object를 가르키는 포인터
- 따라서 대부분 heap allocated object 대상
- global pointer가 local variable을 가르킬 때 dangling pointer 될 수 있음(함수가 종료될 때 스택프레임이 사라지게되어  가르키는 object가 미존재)
- 
    