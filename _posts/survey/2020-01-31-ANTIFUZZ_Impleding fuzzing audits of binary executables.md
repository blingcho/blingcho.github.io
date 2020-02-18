---
layout: post
title: "AntiFuzz: Impeding Fuzzing Audits of Binary Executables"
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
- 기존 fuzzer들을 분석하여 공통된 assumptions들을 세우고 이를 파개하는 방식의 approach
  
___

# Fuzzing Assumptions
<br>

fuzzer들은 최소 다음중 하나의 요구사항이 필요

**1. Coverage Yields Relevant Feedback**
- coverage guided fuzzing
- static/dynamic instrumentation code을 삽입하거나 hardware tracing을 통해 얻은 coverage information을 feedback으로 활용하여 다음 input 생성

**2. Crush Can Be Detected**
- fuzzing 중에 crush가 나는 것을 감지 가능

**3. Many Executions per Second**
- 많은 input들이 실행됨
- 실행이 적으면 feedback이 적어 coverage를 늘려나가기 어려움

**4. Constraints Are Solvable with Symbolic Execution**
- constraints, magic byte 같은 경우는 solver를 이용하여 해당 값 도출
- 이러한 symbolic execution의 경우 solver를 통해 input값으로 어떤 값이 들어가야하는지 계산

___
# Solution
<br>

**1. Attacking Coverage-guidance**
- fake code를 삽입해서 모든 input에 대해 새로운 basic block을 방문하게함
- python 스크립트로 c source 시작 부분에 임의의 edge 생성 
- antifuzz_init()함수를 처음에 호출해야됨

**2. Preventing Crash Detection**
- signal handler로 signal intercept
- ptrace디버거 부착

**3. Delaying Execution**
- 에러시 실행되는 함수에서 복잡한 연산을 실행시켜 실행 지연

**4. Overloading Symbolic Execution Engines**
- solver가 constraints에 대해 역연산이 어렵도록 sha512와 aes-ecb 연산 후 comparison
~~~c
 rst = aes_encryption(text: input,key: sha512(input));
 input aes_decryption(text: rst, key: sha512(input));

 if(input ~~~)
~~~

___
# limitation
<br>

**1. Manual modifying**
- 아무리 적은 양(author는 4~10분만에 한다고함)이라고 해도 수작업으로 코드 수정은 어려움
- 특정 몇몇 작업은 개발자가 antifuzz 코드를 특정 코드에 삽입시켜주는 방식(개발자가 보호할 코드들을 설정해야함/e.g. solution #3, #4)

**2. 바이너리에 constraints 알 수 있음**

~~- 실제 바이너리에는 constraints가 있기 때문에 공격자가 이를 확인하여 임의로 시드 파일에 해당 input을 넣어줄 수 있음~~
- hash comparison을 사용해서 바이너리에는 hash value를 기록함.  

**3. Control-flow corruption**
- 뇌피셜이지만 signal intercept를 통해 그냥 exit, err 발생시 복잡한 연산을 한다는 것 자체가 프로그램의 semantic을 손상시키는 것 같음