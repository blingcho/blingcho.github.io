---
layout: post
title: "FuzziFication: Anti-Fuzzing Techniques"
categories:
  - Fuzzing
tags:
  - security
  - fuzzing
  - survey
  - skimming
---

퍼징은 attacker, defender 모두 효과적으로 이용 가능. 따라서 anti fuzz solution을 통해 defender들은 attacker보다 advantage를 얻고 attacker는 release된 binary executable에 대해 퍼징 못하게 함.

___
# Motivation
<br>

- 최근 취약점 찾기는 사람이 직접 찾는 것보다는 자동화
- 즉, 퍼징이 많이 사용되고 다양한 방법론이 제시되는 중
- 이러한 상황에서 공격자들의 테스트를 어렵게 하여 개발자들이 테스트하는 퍼징보다 성능이 떨어지게 하여 방어
- Anti-Fuzz 논문과 같은 시기에 발표되었고, 접근법도 유사
___

# Fuzzification techniques(anti-fuzz)
<br>


**1. SpeedBump**
- valid input과 fuzzing을 통해 자주 사용되는 path와 error handle 같은 일반적인 상황에서 자주 사용되지 않는 cold path를 분리 
- cold path에 artithmetic한 delay를 삽입, 이 과정에서 data flow dependency를 가지기 위해 프로그램 semantic에 영향을 주지 않는 data flow 삽입(pre-analysis로 speed bump를 제거하지 못하게 하기 위해서)
- developer가 configurable하게 overhead 적용 가능 본 논문에서는 약 2% 이하로 overhead 설정

**2. BranchTrap**
- 실행이 많이 되는 함수의 argument에 sensitive하게 다른 basic block을 실행 
- jmp table[arg1 ^ arg2]로 점프함
- basic block은 다음과 같은 내용으로 함수의 epilogue 기능
~~~x86asm
pop rbp
pop r15
ret
~~~
- 이런 방식으로 함수 return 시 마다 다른 basic block을 통해서 return 하게 되면 모든 feedback이 유용하다고 판단할 뿐만 아니라 fuzzer가 가지고있는 fuzzing state에 collision이 많이 발생해서 coverage guided를 어렵게 할 수 있음.(collision을 해결한 fuzzer가 있지만, 소스코드가 필요하고 최소한 저장할 수 있는 max storage가 존재)

**3. AntiHybrid**
- data-flow dependencies를 implicit하게 transform
- implicit이란 copy 같은 경우 비트 단위로 쪼개서 copy하는 것을 의미하는 것으로 보임
~~~c
 ---explicit---
 int a = b;
 
 ---implicit---
 int a;
 for(int i = 0 ; i < integer_bit_size ; i++){
   a = (b) & (0x1 << i);
 }

~~~
- compare instruction의 operand를 hash 값으로 변환하여 역연산 불가능하게 함

___
# limitation
<br>

**1. Manual modifying**
- anti-fuzz와 다르게 manually하게 코드를 수정하는 작업은 없지만, developer가 직접 어떤 line에 삽입할지 line number를 입력해야 함
  
