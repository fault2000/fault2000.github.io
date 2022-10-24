---
layout: post
title: LLVM pass 종류
category: [llvm, security]
tags: [llvm, pass, optimization, tutorial]
fullview: true
comments: true
author: fault2000
---

Pass의 사용되는 class의 종류에 대해 정리하는 시간을 가져보았다.  

이 글은 공식 문서의 내용을 적당히 번역한 내용임을 알린다. **[LLVM 공식문서 링크](https://llvm.org/docs/WritingAnLLVMPass.html)**<br>
목차는 다음과 같다.
1. ImmutablePass
2. ModulePass
3. CallGraphSCCPass
4. FunctionPass
5. LoopPass
6. RegionPass
7. MachineFunctionPass
<br>
새 pass를 만들때 가장 먼저 해야할 것은 내가 만들 패스를 위해 어떤 클래스를 서브클래스로 지정해야하는가 이다. 우리가 전 포스트에서 다뤘던 pass는 FunctionPass 클래스를 사용했다. 하지만 이러한 사용이 언제, 어떻게 사용되어야 하는지는 다루지 않았다.<br>
이제 사용가능한 class들을 가장 보편적인 것부터 가장 자세한 것까지 다루어보자.<br>
<br>
우리의 pass를 위해 슈퍼클래스를 고를 때, 나열된 요구 사항을 충족할 수 있으면서 가능한 가장 자세한 클래스를 골라야 한다. 이것은 컴파일러가 불필요하게 느려지지 않도록 패스 실행 방법을 최적화하는데 필요한 LLVM 패스 인프라 정보를 제공한다.<br>
<br>
<h2>ImmutablePass Class</h2>
가장 평범하고 지루한 패스 유형은 *ImmutablePass* class이다. 이 pass는 실행하지 않아도 되고, 상태가 바뀌면 안되고, 업데이트가 될 필요가 없는 pass들에게 쓰인다. 이건 변형 혹은 분석의 일반적인 유형은 아니지만, 현재 컴파일러 설정에 대한 정보를 제공한다.<br>
이 pass는 자주 사용되어지지 않음에도 불구하고, 현재 컴파일 중인 목표 머신에 대한 정보와 다양한 변환에 영향을 미칠 수 있는 기타 정적 정보를 제공하므로 중요하다.<br>
ImmutablePass들은 다른 변환을 무효화하지 않고, 무효화되어지지도 않으며, "실행"하지도 않는다.<br>
<br>
<h2>ModulePass Class</h2>
modulePass 클래스는 우리가 사용할 수 있는 모든 상위 클래스 중에서 가장 보편적이다.<br>
ModulePass로 부터 파생되었다는 것은 전체 프로그램을 하나의 단위로 사용하며 이는 함수 본체를 예측불가능한 순서로 참조하거나, 함수를 추가하거나 제거함을 나타낸다.<br>
ModulePass 하위클래스의 행동에 대해서 아무것도 알려진게 없으므로, 이들의 실행동안 최적화는 이루어지지 않는다.<br>
<br>
ModulePass는 getAnalysis 인터페이스인 getAnalysis`<DominatorTree>`(llvm::Function *)를 통해 만약 함수 패스가 아무 모듈이나 immutable 패스를 필요로 하지 않는다면 검색 결과를 함수에게 제공하는데 사용된다.<br>
염두해두어야 할 점은 분석이 실행되어지는 함수에서만 된다는 점이다. 예를 들어, dominators의 경우 선언이 아닌 오직 함수 정의를 위해서만 DominatorTree를 호출할 수 있다는 것이다.<br>
<br>
정확한 ModulePass 하위 클래스를 작성하기 위해서, ModulePass에서 파생되고 다음 문단의 소개되는 특징을 사용하여 runOnModule 메서드를 오버로드한다.

<h4>runOnModule Method</h4>
```c++
virtual bool runOnModule(Module &M) = 0;
```
runOnModule 메소드는 패스에 흥미로운 일을 수행한다. 메소드는 모듈이 변환에 의해 수정된 경우 true를 반환하고 그렇지 않은 경우 false를 반환한다.

<h2>CallGraphSCCPass Class</h2>
*여기서 언급되는 SCC는 strongly connected component의 약자로 유향 그래프 내의 모든 정점이 모든 다른 정점에 대해 도달 가능한 경우 강하게 연결되었다고 표현하며, 강한 연결 요소는 부분 그래프의 모든 정점이 강하게 연결된 임의의 유향그래프를 의미한다.다음 그림은 강한 연결 요소를 표시한 모습을 보여주고있다.*<br>
![image](https://user-images.githubusercontent.com/73513005/150279675-3b295378-23e3-4844-ad39-f18cb33117e1.png)
<br>
CallGraphSCCPass는 호출 그래프(호출자보다 먼저 계산되는)에서 프로그램을 바텀-업 방식으로 순회해야 하는 패스에 의해 사용된다.<br> CallGraphSCCPass에서 파생되는 것은 CallGraph를 만들고 순회하는 몇 가지 메커니즘을 제공할 뿐만 아니라, 시스템이 CallGraphSCCPass들의 실행을 최적화할 수 있도록 허용한다. <br>
만약 당신의 패스가 아래에 요약된 요건을 충족하고 FunctionPass의 요건을 충족하지 못했다면, 당신의 패스는 CallGraphSCCPass에서 파생되어야 한다.<br>
<br>
분명히 말하자면, CallGraphSCCPass 하위 클래스는

1. ... 현재 SCC와 직접 호출자, SCC의 피호출자 이외의 모든 함수들을 검사하거나 수정하는 것은 허용되지 않는다.
2. ... 현재 CallGraph 객체를 보존하고 프로그램에 의해 반영되는 모든 변화들을 갱신해야 한다.
3. ... 현재 모듈에서 SCC를 추가하거나 제거할 수 없다. 단, SCC의 내용은 변경될 수 있다.
4. ... 현재 모듈으로 부터 전역 변수를 더하거나 제거할 수 있다.
5. ... runOnSCC 호출(전역 데이터 포함)에 걸쳐 상태를 유지할 수 있다.

CallGraphSCCPass 구현은 SCC 속 하나 이상의 노드를 포함한 SCC를 처리해야 하기에 약간 까다로운 경우가 있을 수 있다.<br>
아래 문단에 명시된 모든 가상 메소드는 만약 그들이 프로그램을 수정했다면 true를 반환할 것이고, 아니라면 false를 반환할 것이다.<br>

<h4>doInitialization(CallGraph &) method</h4>
```c++
virtual bool doInitialization(CallGraph &CG);
```
doInitialization 메소드는 CallGraphSCCPass들이 허용받지 못하는 일들 중 대부분을 할 수 있다. 이들은 함수를 추가하거나 제거할 수 있고, 함수에 대한 포인터를 얻을 수 있는 등 다양한 일을 할 수 있다.<br>
doInitialization 메소드는 처리 중인 SCC에 의존하지 않는 간단한 초기화 유형을 수행하도록 설계되었다.<br>
doInitialization 메소드 호출은 다른 패스 실행과 겹치지 않도록 계획되어진다.(따라서 호출은 매우 빨라야 한다)

<h4>runOnSCC method</h4>
```c++
virtual bool runOnSCC(CallGraphSCC &SCC) = 0;
```
runOnSCC 메소드는 패스에 흥미로운 일을 수행한다. 메소드는 모듈이 변환에 의해 수정된 경우 true를 반환하고 그렇지 않은 경우 false를 반환한다.(위에 runOnModule과 같다.)

<h4>doFinalization method(CallGraph &CG)</h4>
```c++
virtual bool doFinalization(CallGraph &CG);
```
doFinalization 메소드는 패스 프레임워크가 컴파일 중인 프로그램의 모든 SCC에 대해 runOnSCC 호출을 완료했을 때 호출되는 자주 사용되지 않는 메소드이다.

<h2>Function Pass</h2>
ModulePass의 하위 클래스와는 반대로, FunctionPass 하위클래스는 시스템의 의해 예상할 수 있는 예측 가능하고 지역적인 동작을 가진다.<br>
모든 FunctionPass는 프로그램 속 각 함수에서 프로그램 속 모든 다른 함수들과 독립적으로 실행된다.<br>
FunctionPass들은 특정 순서에 맞춰 실행될 필요가 없으며, 외부 함수에 의해 수정되지 않는다.<br>
<br>
명백히 하자면, FunctionPass 하위 클래스들은 다음이 금지되어 있다.

1. 현재 실행되어지는 함수를 제외한 다른 함수 검사 혹은 수정
2. 현재 모듈에서 함수 추가 혹은 제거
3. 현재 모듈에서 전역 변수 추가 혹은 제거
4. runOnFunction 호출 동안 상태 유지(전역 변수를 포함)

FunctionPass를 구현하는 것은 보통 직관적이다.<br>
FunctionPass는 그들의 일을 하기 위해 세 가지 가상 메소드를 오버로드한다. 이 메소드들은 그들이 프로그램을 수정하면 true, 아니면 false를 반환한다.

<h4>doInitialization(Module &M) method</h4>
```c++
virtual bool doInitialization(Module &M);
```
doInitialization 메소드는 FunctionPass들이 허용받지 못하는 일들 중 대부분을 할 수 있다. 이들은 함수를 추가하거나 제거할 수 있고, 함수에 대한 포인터를 얻을 수 있는 등 다양한 일을 할 수 있다.<br>
doInitialization 메소드는 처리 중인 함수에 의존하지 않는 간단한 초기화 유형을 수행하도록 설계되었다.<br>
doInitialization 메소드 호출은 다른 패스 실행과 겹치지 않도록 계획되어진다.(따라서 호출은 매우 빨라야 한다)<br>
<br>
이 메소드 사용의 좋은 예시는 LowerAllocations 패스다. 이 패스는 malloc과 free 명령어를 플랫폼 의존적인 malloc과 free 함수 호출로 변환한다. 이건 doInitialization 메소드를 통해 필요한 malloc과 free 함수의 참조를 가져오고 필요한 경우 모듈에 프로토타입을 추가한다.<br>

<h4>runOnFunction method</h4>
```c++
virtual bool runOnFunction(Function &F) = 0;
```
runOnFunction 메소드는 당신의 패스에 변형 혹은 분석 작업을 하기 위해 당신의 하위 클래스에서 구현되어야 하는 메소드이다. 보통은, 함수가 수정되었다면 true를 반환한다.

<h4>doFinalization(Module &M) method</h4>
```c++
virtual bool doFinalization(Module &M);
```
doFinalization 메소드는 패스 프레임워크가 컴파일되는 프로그램에서 모든 함수에 대한 runOnFunction 호출을 끝냈을 때, 호출되는 자주 사용되지 않는 메소드이다.

<h2>LoopPass Class</h2>
모든 LoopPass는 함수 속 각 루프마다 실행되며 그 실행은 함수 속 모든 다른 루프들과 독립적이다. LoopPass는 루프 네스트 순서로 루프를 처리하므로, 외부 루프가 가장 늦게 처리된다.<br>
LoopPass 하위 클래스들은 LPPassManager 인터페이스를 통해 루프 네스트를 갱신할 수 있도록 허락한다.<br>
loop pass를 구현하는 것은 보통 직관적이다. LoopPass들은 그들의 일을 하기 위해 세 가지 가상 메소드들을 오버로드하며 이 모든 메소드들은 그들이 프로그램을 수정했다면 true, 아니면 false를 반환한다.<br>
<br>
주 루프 패스 파이프라인의 일부로 실행되도록 설계된 LoopPass 하위 클래스는 그것의 파이프라인 속 다른 루프 패스들이 필요로 하는 모든 같은 함수 분석을 보존할 필요가 있다. 이것을 쉽게 하기 위해서, getLoopAnalysisUsage 함수가 LoopUtils.h를 통해 제공된다.<br>
LoopUtils.h는 하위클래스의 getAnalysisUsage 재정의 내에서 호출되어 일관되고 올바른 동작을 얻을 수 있다.<br>
비슷하게, INITIALIZE_PASS_DEPENDENCY(LoopPass)는 이 함수 분석 집합을 초기화한다.

<h4>doInitialization(Loop *, LPPassManager &LPM) method</h4>
```c++
virtual bool doInitialization(Loop *, LPPassManager &LPM);
```
doInitialization 메소드는 처리 중인 함수에 의존하지 않는 간단한 초기화 유형을 수행하도록 설계되었다.<br>
doInitialization 메소드 호출은 다른 패스 실행과 겹치지 않도록 계획되어진다.(따라서 호출은 매우 빨라야 한다)<br>
LPPassManager 인터페이스는 함수나 모듈 레벨의 분석 정보에 엑세스하는데 사용해야 한다.

<h4>runOnLoop method</h4>
```c++
virtual bool runOnLoop(Loop *, LPPassManager &LPM) = 0;
```
runOnLoop 메소드는 당신의 패스에 변형 혹은 분석 작업을 하기 위해 당신의 하위 클래스에서 구현되어야 하는 메소드이다. 보통은, 함수가 수정되었다면 true를 반환한다.<br>
LPPassManager 인터페이스는 loop nest를 갱신하는데 쓰여야 한다.

<h4>doFinalization method</h4>
```c++
virtual bool doFinalization();
```
doFinalization 메소드는 패스 프레임워크가 컴파일되는 프로그램에서 모든 함수에 대한 runOnLoop 호출을 끝냈을 때, 호출되는 자주 사용되지 않는 메소드이다.<br>

<h2>RegionPass</h2>
*Region은 다음 그림과 같이 그래프의 분기가 바뀔 수 있는 지점을 뜻하는 듯하다... 정확한지는 모르겠지만.<br>
![image](https://user-images.githubusercontent.com/73513005/150284527-e97d5938-b76a-4e03-a62a-3410d0c877f3.png)<br>
RegionPass는 LoopPass와 비슷하다. 하지만 함수 내의 각 단일 시작 단일 종료 영역에서 실행된다.<br>
RegionPass는 영역을 네스트 순서로 실행하기에 가장 바깥에 위치한 영역이 가장 나중에 실행된다.<br>
<br>
RegionPass 하위 클래스들은 RGPassManager를 통해 영역 트리를 갱신할 수 있도록 허락된다.<br>
당신은 당신만의 영역 패스를 구현하기 위해서 RegionPass의 세 가지 가상 메소드들을 오버로드해야한다. 모든 메소드들은 그들이 프로그램을 수정했다면 true를, 아니라면 false를 반환한다.

<h4>doInitialization(Region *, RGPassManager &RGM) method</h4>
```c++
virtual bool doInitialization(Region *, RGPassManager &RGM);
```
doInitialization 메소드는 처리 중인 함수에 의존하지 않는 간단한 초기화 유형을 수행하도록 설계되었다.<br>
doInitialization 메소드 호출은 다른 패스 실행과 겹치지 않도록 계획되어진다.(따라서 호출은 매우 빨라야 한다)<br>
RGPassManager 인터페이스는 함수나 모듈 레벨의 분석 정보에 엑세스하는데 사용해야 한다.

<h4>runOnRegion method</h4>
```c++
virtual bool runOnRegion(Region *, RGPassManager &RGM) = 0;
```
runOnRegion 메소드는 당신의 패스에 변형 혹은 분석 작업을 하기 위해 당신의 하위 클래스에서 구현되어야 하는 메소드이다. 보통은, 함수가 수정되었다면 true를 반환한다.<br>
RGPassManager 인터페이스는 loop nest를 갱신하는데 쓰여야 한다.

<h4>doFinalization method</h4>
```c++
virtual bool doFinalization();
```
doFinalization 메소드는 패스 프레임워크가 컴파일되는 프로그램에서 모든 함수에 대한 runOnRegion 호출을 끝냈을 때, 호출되는 자주 사용되지 않는 메소드이다.<br>

<h2>MachineFunctionPass Class</h2>
MachineFunctionPass는 프로그램 내의 각 LLVM 함수들을 기계-의존적인 표현에서 실행되는 LLVM 코드 생성기의 일부이다.<br>
코드 생성기 패스는 특별히 TargetMachine::addPassesToEmitFile 및 이와 비슷한 경로에 의해 등록되고 초기화되어진다. 따라서 일반적으로 opt 혹은 bugpoint 명령에서 실행할 수 없다.<br>
<br>
MachineFunctionPass는 또한 FunctionPass이기도 하다. 그러므로 FunctionPass에 적용되는 모든 제한 또한 적용된다.<br>
MachineFunctionPass들은 추가 제한을 가지고 있다. 특별히, MachineFunctionPass들은 다음과 같은 사항이 금지되어 있다.

1. 모든 LLVM IR 명령어, BasicBlocks, 인자, 함수, 전역변수, 전역가명(GlobalAliases), 모듈을 수정하거나 만들기
2. 현재 실행되어지는 MachineFunction을 제외한 다른 MachineFunction을 수정하기
3. runOnMachineFunction 호출 동안 상태 유지(전역 변수를 포함)

<h4>runOnMachineFunction(MachineFunction &MF) method</h4>
```c++
virtual bool runOnMachineFunction(MachineFunction &MF) = 0;
```
runOnMachineFunction은 MachineFunctionPass의 주요 시작 지점으로 간주할 수 있다. 즉, 당신은 자신만의 MachineFunctionPass의 일을 수행할려면 이 메소드를 재정의를 해야한다.

튜토리얼에서 소개된 클래스들은 전부 살펴보았다. 다음 포스트에서는 튜토리얼에서 패스를 등록하는 과정에 대한 설명을 정리해보도록 하겠다.