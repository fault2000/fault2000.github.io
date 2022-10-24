---
layout: post
title: LLVM pass 등록
categories: [llvm]
tags: [llvm, pass, optimization, tutorial]
fullview: true
comments: true
author: fault2000
---

우리가 전에 보았던 Hello World 패스 예시에서 우리는 패스 등록이 작동하는 방법을 조명했고 어떻게 쓰이는 지와 무엇을 하는 지에 대한 이유를 설명했다.  
여기서 우리는 패스가 등록되는 것이 왜 어떻게 되는지 논의할 것이다.  

위에서 봤듯, 패스들은 RegisterPass 템플릿과 같이 등록된다. 템플릿 파라매터는 프로그램(예를 들어, opt 혹은 bugpoint)에 추가할 패스들을 특정하기 위해 명령줄에 사용되는 패스의 이름이다.  
그 첫 인자는 패스의 이름으로, 프로그램의 -help 출력에 쓰이며, 또한 -debug-pass 옵션에 의해 생성되는 디버그 출력에도 쓰인다.  
  
만약 당신이 자신의 패스가 쉽게 덤프되기를 원한다면, 당신은 아래와 같은 print 메소드를 구현해야 한다.  

#### print method

```c++
virtual void print(llvm::raw_ostream &O, const Module *M) const;
```

print 메소드는 분석 결과를 인간이 읽을 수 있는 버전으로 출력해주기 위해 "analysis"로 구현되어야 한다.  
이것은 분석 그 자체로 디버깅에 유용할 뿐만 아니라 다른 사람들이 분석이 어떻게 작동하는지 알아내기에도 유용하다. opt의 -analysis 인자를 사용하여 이 메소드를 불러올 수 있다.  
  
llvm::raw_ostream 매개 변수는 결과를 기록할 stream을 지정하고, 모듈 매개 변수는 분석된 프로그램의 최상위 모듈에 포인터를 제공한다.  
그러나 염두할 점은 이 포인터가 특정 상황에서는 NULL일 수도 있다는 것이다.(디버거로 부터 Pass::dump()를 호출할 때), 그러므로 포인터를 오직 디버그 결과를 향상시키는데 써야하지 의존하면 안된다.  

## Specifying interactions between passes

PassManager의 주요 책임들 중 하나는 패스들이 다른 패스들과 제대로 상호작용하도록 보장하는 것이다.  
PassManager가 패스들의 실행을 최적화하려고 시도하기 때문에, 패스들이 어떻게 서로간의 상호작용을 하는지, 여러 패스들 중 의존성이 있는지에 대해 알아야 한다.  
이것들을 추적하기 위해서, 각 패스는 현재 패스 이전에 실행되어야 하는 패스 집합과 현재 패스에 의해 무효화된 패스를 선언할 수 있다.  
  
보통 이 기능은 패스를 실행하기 전에 분석 결과를 계산하도록 요구하는데 사용된다. 임의 변환 패스를 실행하는 것은 무효화 집합이 지정하는 계산된 분석 결과를 무효화할 수 있다. 만약 패스가 getAnalysisUsage 메소드를 구현하지 않았다면, 기본적으로 필수 패스를 가지지 않으며, 다른 모든 패스를 무효화한다.  

### getAnalysisUsage method

```c++
virtual void getAnalysisUsage(AnalysisUsage &Info) const;
```

getAnalysisUsage 메소드를 구현하면 변환에 필요한 집합과 유효하지 않은 집합을 지정할 수 있다. 구현은 AnalysisUsage 객체에 어떤 패스가 필요하고 무효화되지 않는지에 대한 정보를 입력해야 한다. 이것을 하기 위해서 패스는 AnalysisUsage 객체에 다음과 같은 메소드들을 호출하여야 한다.

### AnalysisUsage::addRequired<> and AnalysisUsage::addRequiredTransitive<> methods

만약 당신의 패스가 실행되기 위한 이전 패스가 필요하다면, 다음 메소드들 중 하나를 사용함으로써 당신의 패스가 실행되기 전에 필요한 패스를 배치할 수 있다.  
LLVM은 DominatorSet에서 BreakCritivalEdges에 이르는 것을 필요로 할 수 있는 다양한 유형의 분석과 패스들을 가지고 있다. 예를 들어 BreakCriticalEdges를 요구하면 당신의 패스가 실행될 때, CFG에 critical edge가 없음을 보장한다.  
  
몇 분석들은 그들의 일을 하기 위해 다른 분석과 연계한다. 예를 들어, AliasAnlysis`<aliasAnalysis>` 구현은 다른 별칭 분석 패스들과의 연계를 필요로 한다. 분석 체인의 경우 addRequired 메소드 대신 addRequiredTransitive 메소드를 사용해야 한다. 이것은 패스매니저에게 필요되어지는 패스가 필요한 패스가 존재하는 한 활성 상태여야 함을 알린다.  

### AnalysisUsage::addPreserved<> method

PassManager가 하는 일들 중 하나는 분석이 실행되는 방법과 시기를 최적화하는 것이다. 특히, 패스매니저는 필요하지 않는 한 데이터 재입력을 피하려 한다. 이러한 이유 때문에, 패스들은 만약 가능하다면 기존 분석을 보존(즉, 무효화하지 않음)한다고 선언할 수 있다.
