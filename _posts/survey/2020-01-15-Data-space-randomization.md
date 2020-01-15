---
layout: post
title: "Data Space Randomization"
categories:
  - Memory Corruption
tags:
  - security
  - survey
  - skimming
---

Address Space Layout Randomization에서 32bit머신에서 entropy가 낮은 문제와 information leak에 대응하기 위한 방법론

___
# Motivation
<br>

- memory corruption을 막기위해 다양한 randomization 기술들이 등장하지만 각각 한계가 존재
- ASLR은 informaiton leak을 통해 randomization된 layout구조를 알 수 있음
- 또한 2^32bit machine에서는 entropy가 낮음
  
___

# Idea
<br>

- 기본적으로 source to source transformation을 사용
- expression, statement에서 variables에 특정 xor masking 코드를 삽입
~~~c
int x, y, z, *ptr;
...
ptr = &x;
...
ptr = &y;
...
L1: z = *ptr;
~~~
아래와 같이 바뀜
~~~c
int x, y, z, *ptr;
...
ptr = (&x) ^ (m_ptr);
...
ptr = (&y) ^ (m_ptr);
...
L1: z = ((*ptr) ^ (m_ptr)) ^ m_z;
~~~
- 즉 read, write를 할때마다 random variable을 masking(encryption, decryption)함

___
# Key value for masking
<br>

- 프로그램이 시작할 때와 stack,  heap에 변수가 할당될 때에 랜덤변수를 생성
- points-to analysis를 이용하여 한 포인터가 가르키는 object들의 경우 같은 value(key)를 사용

___
# limitation
<br>

### **Overhead**
- read/write할때마다 xor 연산을 하게되므로 overhead가 클거라고 예상됨
- 본 논문에서는 memory region을 나눠서 simple local variable(non-overflow candidates)를 떨어뜨려놓고 xor 연산을 하지않는 방식으로 optimization을 진행
- 실제로는 평균 15%의 overhead
### **source transformation **
- 논문에서도 mismatch가 발생할 수 있고(CIL을 사용하여 하나로 merge한 후 transformation을 진행하였다고 서술), 해당 부분을 manually 수정해야된다고 함
- 컴파일러로 구현은 불가능한가?

