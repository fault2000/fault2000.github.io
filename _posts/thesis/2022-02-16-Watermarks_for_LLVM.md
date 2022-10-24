---
layout: post
title: Digital Watermarks for LLVM Intermediate Representation 리뷰
categories: [thesis]
tags: [thesis, watermark, LLVM]
fullview: true
comments: true
use_math: true
author: fault2000
---

## INTRODUCTION

워터마크를 소프트웨어 속에 적용할 수 있는 방법 소개

## RELATED WORKS

디지털 워터마크를 소프트웨어에 적용하기 위해 두 가지 기존 접근 방식이 있다.  
첫번째는 프로그램 바이너리를 수정하여 워터마크를 박아넣는 방법이다. 다른 방식은 소스 코드를 수정하여 워터마크를 박아넣는 방법이다. 그러나 위 방식들은 최적화를 고려하지 않고, C++ 언어 특징 때문에 매우 작은 워터마크 용량을 얻을 수 있다.  
또한 자바 소스 코드와 바이트코드의 변화를 합친 방식이 사용되는데, 가짜 메소드를 추가하여 실행이 되진 않지만, 워터마크를 박아넣을 수 있는 공간을 만들고, 메소드 속 instruction이나 operand에 정보를 넣는다.  
이 방식은 더미 메소드를 크게 만듬으로써 임의의 길이의 워터마크를 넣을 수 있다는 장점이 있다. 또한 기호 이름이 바뀌는 간단한 난독화에 저항하기도 한다. 단, 이러한 방식은 디컴파일 결과의 부자연스러움으로 인해 내장 위치가 쉽게 확인될 수 있다. 게다가, 더미 메소드의 명령들이 실제 프로그램의 행동과 관련이 없다보니, 만약 내장 위치가 알려지면 내용이 쉽게 덮어씌어질 수 있다.  
또한 이 방식은 프로그래밍 언어의 특징과 머신 코드의 포맷에 의존한다.

## EMDEDDING METHODS

삽입은 다음 세 가지 방법으로 실현된다.

- Method-1: basic block의 순서 변경
- Method-2: instruction operand의 위치 변경
- Method-3: 함수의 순서 변경

### Method-1

첫 번째 방법은 같은 함수 내의 basic block들의 순서를 변경하는 것이다. basic block는 언제나 jump instruction으로 종료되므로, 시작 block을 제외한 나머지 블록의 순서를 변경해도 문제가 없다.  
이러한 방법으로 담을 수 있는 정보의 양은 함수 속 basic block의 수와 비례한다. A를 함수 속 basic block의 수라고 가정하면, 워터마크의 용량은 총 $log_2(A-1)!$ bit이다.

### Method-2

보통 LLVM IR instruction은 3-address code와 비슷하다. 이는 2개의 입력 operand과 1개의 출력을 가짐을 의미한다. add, imul, xor들은 **순서에 영향을 받지 않는다.**. 이는 출력이 operands 순서 변화에 영향을 받지 않음을 의미한다. 이를 이용해, 두 번째 방법은 순서에 영향을 받지 않는 바이너리 instruction의 두 operand의 자리를 바꿔 디지털 워터마크를 새기는 것이다.  
또한, icmp, fcmp 같은 비교 instruction은 비교 sign의 방향을 바꿈으로써 두 operand의 자리를 바꿀 수 있다.  
이 두번째 방법의 정보 용량은 순서에 상관없는 바이너리 instruction과 비교 instruction의 수와 비례한다. B가 그러한 instruction들이라 할 때, 워터마크 용량은 B bit가 될 것이다.

### Method-3

3번째 방법은 모듈 내의 함수들의 순서를 바꾸는 방식이다. 함수의 순서를 바꾸면 그에 상응하는 머신 코드 속 함수의 출력도 같은 순서로 변경된다. 워터마크는 함수의 순열로 표현된다.  
LLVM IR을 조작함으로써, 우리는 모듈 속 정의된 함수의 순서를 자유롭게 바꿀 수 있다. 하지만 모듈 속 먼 거리로 함수를 교환할 경우 캐쉬 손실이 일어날 수 있다. 따라서, 순서 변경은 근접한 함수 그룹에서만 일어나도록 제한해야 한다.  
$G_i$(i=1,...,n)를 모듈 속 함수 그룹이라고 해보자, 그리고 $C_{G_i}$를 그룹 속 함수의 수라고 가정하자. 워터마크의 용량은 $\sum_{i=1}^nlog_2C_{G_i}!$ bit가 될 것이다.  
  
이 세가지 방법들은 서로에 대해 비의존적이기 때문에, 다양한 방법을 동시에 적용할 수 있다.

## RESISTANCE TO OPTIMIZATION

각 방법은 IR에 적용되기에, 중간 표현 수준의 최적화의 효과가 무시될 수 있다. 하지만, 머신 코드 최적화와 링크 타임 최적화는 우리의 워터마크 정보를 파괴할 수 있다. 다음은 이러한 최적화에 대한 워터마크 영향 여부를 고려할 것이다.  
머신 코드 최적화는 instruction을 교체하거나 basic block을 재배치하여 실행 효율성을 증진시킨다. 즉, method-1,2를 잃어버릴 가능성이 있다. 대조적으로, Method-3은 해당사항이 없으므로 손실되지 않는다.  
링크 타임 최적화는 함수를 줄이거나, 제어 흐름을 바꾸므로, 모든 방법을 잃어버릴 수 있다.

## PERFORMANCE EVALUATION

성능 결과는 기존 프로그램과 큰 차이가 있지 않음을 보여주고 있다. 생략