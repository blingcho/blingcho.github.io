---
layout: post
title: "SoK: Eternal War in Memory"
categories:
  - Summary
tags:
  - security
  - survey
---

## ***SoK: Eternal War in Memory***

유형별 메모리 공격 기법과 보호 기법 소개 및 정리한 페이퍼

### **Attack**
 A. Memory Corruption
    
    memory corruption은 보통 2단계로 구성된다(pointer 관련되서는).  

    1. 포인터가 invalid하고 
    2. 해당 포인터를 dereference.
  
    out-of-bounds 된 포인터는 a spatial error. 
    free된 영역을 가르키는 포인터는 a temporal error.

    

    