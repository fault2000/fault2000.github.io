---
layout: post
title: SaViorL, Thwarting Stack-Based Memory Safety Violations by Randomizing Stack Layout 리뷰
category: [thesis]
tags: [thesis, stack-based memory, randomizing]
fullview: true
comments: true
use_math: true
author: fault2000
---

SaVioR 논문 전편은 **[링크](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9369900)**에서 찾아볼 수 있다.<br>
이 논문에서는 공간적(spatial), 일시적(temporal)이라는 용어가 자주 언급되는데, 공간적은 메모리 공간을 공격하는 방식으로 주로 buffer overflow가 대표적인 예이며, 일시적은 메모리 free와 같은 일시적인 타이밍을 노리는 공격으로 주로 use after free의 예시를 가진다.

<h3>INTRODUCTION</h3>
 call stack은 높은 공간적, 시간적 지역성을 가지고 있어 공격자가 공격 자주 활용하며, 이는 heap보다 stack 기반 공격이 더 선호되게 한다. stack 기반 공격 방어 기술에는 크게 4가지로 나누어진다.
 {% highlight yaml %}
 1. canary-based: 낮은 cost, 높은 호환성 단 canary의 비밀 값이 들통나면 무효화
 2. shadow stack-based: 버퍼 오버플로우로부터 return 주소 보호, 단 다른 변수 보호 불가
 3. randomization-based: 패딩 추가로 stack frame layout에 랜덤성 추가, 중요한 버퍼 랜덤 지정, stack 지역의 기반 주소 랜덤화 등으로 공격자의 예측 어렵게 하지만 이러한 방식은 stack frame의 순서를 바꾸지 않아 결국 충분치 않음.
 4. comprehensive protection
{% endhighlight %}
 위 1,2,3번의 방식은 부분적 방어만을 제공하며, 특히 공간적(버퍼 오버플로우같은 메모리 공간을 덮어쓰는 방식) 공격만을 방어하고 일시적(use after free같은 free되는 타이밍을 노리는 방식) 공격이나 진보된 공간적 공격을 막지 못한다.<br>
 공간 및 시간적 공격을 모두 해결하기 위해, 포괄적인 무작위화 기술은 stack 상주 개체(stack 프레임과 stack 프레임 자체에 취약한 버퍼)를 무작위로 할당한 다음 stack 프레임 재사용의 예측 가능성을 줄임으로써 stack 레이아웃의 예측 가능성을 낮추는데 초점을 맞추지만, 큰 성능 overhead, 현존하는 software와의 낮은 호환성을 가진다.
 
 위 문제점을 해결하기 위해 이 논문에서는 **SaVioR**을 소개한다, stack 레이아웃에 예측 불가능한 기능을 도입하는 효율적이고 포괄적인 stack 보호 메커니즘이다.<br>
 더 자세하게는, *SaVioR*는 64-bit 가상 주소 공간이 매우 크며, 띄엄띄엄 분산되어 있는 여러 stack을 만든다는 것을 이용한다. 그러면, *SaVioR*는 각 함수 호출마다 stack 상주 개체와 stack frame 자체를 여러 stack들 중 하나에 무작위로 배치한다. 특히, *SaVioR*는 두 가지 기술을 소개한다. Vulnerable Buffer Isolation(VBI)와 Stack Frame Randomization(SFR)이 그것이다.<br>
 VBI는 spatial 공격에, SFR은 temporal 공격을 방지하는 용도로 사용된다.<br>
 *SaVioR*는 LLVM compiler를 기반으로 만들어졌으며, SPEC CPU2006, PARSEC 3.0 benchmark와 널리 사용되는 실제 응용 프로그램을 통해 평가되었다.

**BACKGROUND & RELATED WORK와 THREAT MODEL은 생략**

<h3>DESIGN</h3>
<h4>VBI</h4>
 VBI는 공격 벡터로 사용될 수 있는 취약한 변수(공간적 메모리 에러를 일으킬 수 있는 모든 stack 상주 객체)들의 주소를 각 함수 호출마다 무작위화하여 목표 변수로부터 공간적으로 격리한다.<br>
 어떻게 VBI가 stack layout을 무작위화하는지는 다음과 같다.<br>
 ![Fig1](https://user-images.githubusercontent.com/73513005/147687936-31472728-1dd3-4292-a6e5-d2b3f0b7fb46.png)<br>
 위 그림은 전체적인 *SaVioR*의 모습이다. 위 단일 Stack이 기존 application의 동작이고, 아래 Stack N개로 이루어진 부분이 SaVioR의 동작을 보여준다.<br>
 Stack 0가 main Stack, Stacks 1-N이 다중 가상 스택들로 구성되어 있다.<br>
 VBI는 모든 취약한 변수들을 무작위로 재배치하고 따라서 목표 변수들을 공간적 메모리 에러로부터 격리시킨다. 예를 들어, <font color='red'>함수 A</font>를 보자. 함수 A는 취약한 버퍼(Vuln buffer)를 가지고 있다. SaVioR가 이 버퍼를 무작위로 선정된 Stack N에 재배치하고, 따라서 return address(R)과 분리된다.<br>
 이를 통해, 취약한 변수는 가치있는 목표들과 공간적으로 근접하지 않게 된다.<br>
 참고할 만한 점은, 취약한 객체의 위치가 무작위임에도 불구하고, SaVioR는 재배치된 객체가 언제나 현재 위치보다 다른 Stack에 재배치됨을 보장한다는 것이다. 이는 메모리 재사용의 가능성을 줄임으로써 공간적인 공격뿐만 아니라, 일시적인 공격까지도 방해한다.
 <h4>SFR</h4>
 VBI가 격리를 통해 취약한 벡터를 격리한다 해도, 다른 Stack 상주 객체(Function pointer, decision-making variable)들을 무작위화하진 않으며 이들은 공격자에겐 매력적인 목표가 될 수 있다.<br>
 VBI를 보완하기 위해, SFR이 도입되었다. SFR은 각 Stack frame에 무작위 크기의 padding을 삽입하고, Stack frame을 비연속적이고 순서가 맞지 않는 방식으로 무작위화한다.<br>
 여기서 스택 프레임 자체를 무작위화하는 것보다 VBI로 모든 변수의 위치를 무작위화하여 이를 구현할 수 있다고 생각할 수 있지만, 모든 변수의 위치를 무작위화한다면 상당한 runtime overhead(난수 생성 증가 및 형편없는 지역성)가 발생한다. 따라서, 모든 변수를 무작위화하는 것보다 목표 가능성이 있는 변수들을 담은 Stack frame의 위치를 무작위화할 것이다.<br>
 더 나아가 SFR은 Stack frame이 비연속적이고 자주 재사용되지 않게 만들어준다. 이는 일시적인 공격의 신뢰성을 떨어트린다.<br>
 예시를 위해 위 그림으로 다시 가보자. 함수 A, B, C의 각 stack frame이 Stack 0, 1, N에 각각 있음을 볼 수 있다. 이처럼, 무작위로 선정된 stack에 stack frame을 위치한 후, 무작위 크기의 padding까지 넣어짐을 볼 수 있다.
 <h4>Identifying Stack Objects to be Randomized</h4>
 성능을 위해서는 스택의 높은 지역성이 필수적이다. SaVioR는 보안을 위해 지역성을 희생할 뿐만 아니라 stack 상주 객체를 무작위화하기 위한 instruction들을 추가하므로 성능 저하를 일으킨다. 따라서, 모든 함수와 스택에 SaVioR를 적용하는 것은 비실용적이다. 즉 우리는 SaVioR를 적용할 객체를 선택하여야 한다.<br>
 ![table1](https://user-images.githubusercontent.com/73513005/147717836-90e724df-ae8c-4d51-9310-a98878ff4fd1.png)<br>
 위 테이블의 빨간색 체크는 각 공격에 사용될 수 있는 취약한 객체를, 초록색 화살표는 공격을 막기 위해 적용되는 방어 기법을, 회색은 부분적으로 적용되는 방어를 의미한다.<br>
 <h4>VBI-applied object</h4>
 여기서는 -fstack-protector-strong와 유사한 정책을 채택하여 취약한 변수를 식별한다. 저 자세히 보자면 SaVioR는 1.아무 타입의 배열 2.배열을 담고 있는 모든 종합적 타입 3.다양한 크기의 배열에 VBI를 적용한다.<br>
 <h4>SFR-applied object</h4>
 주소 추출 변수는 dangling pointer를 일으킬 수 있다. 주소 추출 변수만 있는 frame을 만들어 SFR을 적용한다. 이는 stack frame으로부터 주소 추출 변수들을 분리한다. 이러면 비록 그들끼리는 공간적 공격에 노출되지만, 위 1,2,3번 보다 주소 추출 변수는 공간적 공격을 일으킬 가능성이 적다고 저자는 주장한다.<br>
 SFR의 목표가 일시적 공격을 막는 것인 만큼, 일시적 에러를 일으킬 수 있는 함수에 SFR이 적용된다. 보통 UaF 공격은 공격자가 명백히 관리할 수 있는 heap에 배치된 객체를 목표로 하며 그래서 Stack 기반 UaF 취약점은 드물게 보고된다. 하지만 여기서는 공격 모델의 엄격성을 위해 stack 기반 UaF를 고려한다.<br>
 <h4>Uninitialized Read(UR)</h4>
 선언과 사용은 다양한 함수에서 이루어질 수 있기 때문에, UR을 감지하는 것은 꽤나 도전적이다. 따라서, 현대의 compiler들은 모든 메모리 할당을 zero-initialize(0으로 초기화)한다. 불행히도 최근 출판물에서 이들의 비용이 높은 것으로 나타났고, 이를 대신해 Clang compiler의 -Wunintialized option을 활용하는 것도 한 가지 방법이지만, 형편없는 coverage만을 제공한다. 다음 표에서 여기서 사용되는 감지 도구와 최근 compiler의 분석에 대한 비교를 볼 수 있다.<br>
 ![table3](https://user-images.githubusercontent.com/73513005/147717568-5f0af509-adc6-4a35-b682-de7ac738bb69.png)<br>
 이처럼 SFR만 적용한다면 target 변수와 취약한 변수가 공간적으로 인접하게 되고, VBI만 적용하면 target 변수가 무작위화되지 않은 채로 남겨져 예측 가능하게 된다. 이 둘을 동시에 사용함으로써, 각자의 약점을 보완하고 무작위성을 더 늘리는 효과를 가져올 것이다.
 
 이제, SaVioR가 어떻게 주소 공간을 설정하고, stack layout을 무작위화하는지 알아볼 것이다.
 <h4>Setting Up the Virtual Address Space</h4>
 ![fig2](https://user-images.githubusercontent.com/73513005/147721677-ddd1adf7-48d2-42ae-b360-a208c64ca235.gif)<br>
 위 그림처럼, SaVioR는 두 유형의 영역으로 나뉜다. 하나는 기존 영역, 나머지는 여러 stack-only 영역들이다. 기존 영역은 기존 프로세스의 주소 공간과 비슷하지만, 사용가능한 주소 영역이 b("<"47)bits로 한정되어 있다. 지금부터, 여기선 기존 영역이 42 bits로 제한되어 있다고 가정한다.(뒤에서 설명) 42 bits 위에 남아 있는 주소 공간은 기존 영역의 원래 stack에 해당하는 가상 stack을 가진다.<br>
 <img width="692" alt="fig3" src="https://user-images.githubusercontent.com/73513005/147722095-b481dc0f-bfa3-46d0-9b3f-fcc1d8ca9f17.png"><br>
 위 그림의 a을 보면, 위가 기존 x86_64 리눅스 프로세스의 포인터 표현이다. 크기는 64비트이지만 오른쪽 48 bits만 사용되며, 실제로는 두 부분으로 나뉘어 각각 사용자, 커널을 위해 예약된다. 여기선 오직 유저 주소 영역만 고려한다.<br>
 b를 보면 SaVioR의 포인터 표현을 보여준다. 47 bits 가상 주소 영역이 32개의 42 bits 영역으로 나뉘어진다. 각 영역은 서로 다른 상위 5 bits를 가지며 이는 random bits로 불린다. random bit 0은 기존 영역을 위해, random bits 1-31은 stack-only 영역을 위해 있다. 프로세스 초기화 시 각 stack-only 영역을 위해, SaVioR는 생성된 각 stack이 random bits를 제외하고 일반 stack과 동일한 주소 범위를 갖도록 추가 스택을 생성한다.<br>
 fig2, 위에서 두번째 그림을 보면 설명한대로 구현된 가상 주소 공간 구성을 볼 수 있다.
 <h4>Randomizing Stack Objects using Pointer Mirroring</h4>
 SaVioR는 random bits에 상응하는 여러 스택들을 가지고 있다. VBI나 SFR로 보호되는 함수가 실행되면, stack 상주 객체는 여러 스택들 중 하나로 무작위로 배치된다. 이것은 무작위 stack 객체의 random bits를 무작위로 변경함으로써 성취할 수 있다. 이것을 여기선 *pointer mirroring*이라고 부른다.<br>
 VBI가 적용된 함수에게 SaVioR는 취약한 객체를 무작위화하고 모든 stack access instruction들을 무작위로 재배치된 객체를 참조하도록 수정한다. SFR 보호를 위해선, SaVioR은 stack pointer를 무작위화함으로써 stack frame을 재배치하기 위해 모든 SFR이 적용된 함수를 호출하는 instruction들을 수정한다.
 <h4>Address Space Reduction</h4>
 SaVioR는 가상 주소 공간을 희생하여 무작위화를 제공한다.<br>
 SaVioR는 스택에 추가 랜덤화를 도입하기 때문에, ASLR(Address Space Layout Randomization)과 완벽히 호환되어야 하며 동시에 의존해야 한다. 그러므로, SaVioR은 사용가능한 주소 공간을 줄임으로써 ASLR의 효율성에 영향을 주는 것을 피해야 한다.(SaVioR의 무작위성 엔트로피(stack의 수와 random bits)가 늘어날수록, ASLR의 효율성은 훼손된다.)<br>
 여기서, 우리는 기본적인 ASLR 엔트로피를 제공하기 위해 필요한 사용가능한 주소 공간의 최소 양을 기술할 것이다.<br>
 Linux 커널은 4개의 영역으로 구분된다. 가장 낮은 주소에 위치하는 executable과 brk 영역 가장 높은 주소에 위치하는 stack과 memory mapped 영역이다.<br>
 executable 영역은 ELF와 PIE-enabled 바이너리를 위한 28 bits의 엔트로피를 가진다.<br>
 brk 영역은 보통 heap을 위해 사용되며 13 bits의 엔트로피를 가진다.<br>
 memory-mapped 영역은 라이브러리와 memory-mapped 구역을 가지며 28 bits의 엔트로피를 가진다.<br>
 마지막으로, stack 영역은 22 bits의 엔트로피를 가진다.<br>
 상기해야할 점은, ASLR이 페이지 수준 세분화를 기반으로 하기 때문에, 최소 12 lesat significant bits(최하위 비트)는 고정이라는 것이다. 이것은 n-bit 엔트로피를 위한 주소지정가능한 bits가 0~$2^{n+12}$의 범위를 가진다는 것을 의미한다.<br>
 요약하자면, Linux 속 ASLR의 엔트로피를 지원하기 위해선, 최소한 ELF, heap, memory-mapped, stack segments는 각각 40, 25, 40, 34 bits의 주소지정가능한 주소 공간이 보존되어야 한다.(각 부분의 크기는 고려하지 않는다.) 이것은 주소 공간이 최소 42 bits는 있어야 Linux 내의 보통 수준의 ASLR 엔트로피를 제공할 수 있음을 의미한다.<br>
 주어진 5개의 random bits는 오직 32개의 stack만을 지원하므로, 메모리 에러로부터 안전하다고 여겨질 수 없다. 다행히도, 인텔은 5단계 페이징을 제안했다. 이는 주소지정가능한 주소 공간을 48 bits에서 57 bits 까지 확장해주고, 이는 SaVioR가 14 random bits를 지원가능하게 해주고, 최대 16,384 스택을 가용할 수 있다.

 <h3>IMPLEMENTATION</h3>
 Linux-5.0.8에 SaVioR를 적용하기 위해 LLVM/Clang-8.0.0 컴파일러 프레임워크를 기반을 사용했다.
 <h4>Static Instrumentation in LLVM</h4>
 여기서, VBI와 SFR을 위해 static instrumentation modules이 어떻게 적용되었는지 설명한다.<br>

 <img width="280" alt="image" src="https://user-images.githubusercontent.com/73513005/148163737-2d2073c8-8029-44db-90f9-acf7174d973d.png"><br>

 **Helper Functions.**<br> 위 그림의 line 3의 slr_random은 난수를 생성한다. 이것은 함수가 아닌 LLVMx86의 고유한 동작으로 구현된다. 여기서는 Intel AES-NI 명령어 세트를 기반으로 AES 기반 난수 생성을 구현했다. 생성된 난수와 무작위의 비밀 키는 각각 전용 레지스터인 xmm14와 xmm15에 저장된다.<br>
 난수를 생성하기 위해, slr_random은 하나의 aesenc 명령어를 실행하고 128bit 무작위 값을 가져온다. 생성된 값은 xmm14 레지스터의 상위 64bit와 하위 64bit로 구성되며 이는 각각 pextrq와 movq 명령어로 추출될 수 있다.<br>
 생성된 무작위 값은 VBI와 SFR의 stack 상주 객체를 무작위화하는데 사용된다. 컴파일러 기반 접근의 한가지 장점은 SaVioR이 컴파일러 옵션으로 구현됨으로써 개발자가 선택적으로 SFR과 VBI를 적용할 수 있다는 것이다.<br>
 SFR, VBI가 동시에 활성화될 때, SaVioR은 먼저 SFR을 사용하여 stack frame을 무작위화하고, VBI를 통해 취약한 변수들을 무작위화한다. 이것은 VBI가 random 값을 재생성할 필요없이, SFR이 완료되고 남은 무작위 값을 사용할 수 있게 해준다. 이것은, 하위, 상위 부분의 무작위 숫자가 각각 SFR, VBI에 적용됨을 의미한다.<br>
 line 5의 slr_alloca는 VBI와 SFR instrumentation이 각각 취약한 변수와 stack frame들을 무작위화하는데 사용되는 pointer mirroring 기법을 구현하는데 사용된다.<br>
 이것은 두 인자를 받는다. pointer와 random number, 각각 무작위화해줄 취약한 객체 혹은 stack pointer, 무작위화 될 객체가 지정될 스택을 정해주는데 사용되는 값이다.<br>
 slr_alloca는 random bit를 설정함으로써 stack 중 하나와 ptr을 미러링한다. 자세히 보자면, slr_alloca는 객체 위치를 임의화하도록 강제하며, 취약한 변수들이 원래의 stack frame으로부터 완전히 분리되도록 보장한다. 이를 달성하기 위해, line 7을 보면 rnd 값은 객체가 배치된 현재 스택 숫자와 XOR되어진다.<br>
 **VBI Pass**<br>
 VBI Pass는 stack 변수(타입, 길이)에 대한 정보를 포함하는 LLVM IR의 alloca 명령어에 위치하고, 취약한 변수를 포함하는 모든 함수들을 계측한다.<br>
 <img width="287" alt="image" src="https://user-images.githubusercontent.com/73513005/148170908-27f237ba-ec97-4116-8eb4-a36836b8a961.png"><br>
 VBI instrumentation이 어떻게 작동하는지는 위 그림의 수도 코드를 보면 나와 있다. 왼편을 보자.<br>
 메인 함수의 prologue를 buffer의 위치를 무작위화하게 작성된다. 먼저, 작성된 메인 함수는 무작위값을 얻기 위해 slr_random을 실행한다. 그림의 간단함을 위해, 여기선 slr_random이 1~N 사이의 난수를 돌려준다고 가정했다.(N은 배치된 스택의 수를 나타낸다) 왜냐하면 slr_alloca가 0의 값을 인자로 받게 된다면, 현재 스택 수와 0을 XOR하게 되어 취약한 버퍼를 stack frame으로 부터 분리하지 못하기 때문이다.<br>
 하지만, 실제 구현에서는 LLVM 컴파일러는 사용 가능한 스택 중 하나를 무작위로 선택하는데 필요한 비트를 제외한 random bit의 모든 값을 숨기고 0이 아닌 난수가 slr_alloc에게 넘겨지는 것을 보장하는 명령어를 삽입한다.<br>
 0이 아닌 난수를 통한 무작위화는 다음과 같이 표현 된다. 
![CodeCogsEqn](https://user-images.githubusercontent.com/73513005/148181786-eda9e2ae-809b-44f4-ac9d-a1954c3d93f8.png)
 (rnd는 무작위값을, n은 random bits를 뜻한다.)<br>
 다음으로, slr_alloca는 buf와 rnd를 인자로 실행되어져 rnd_buf(무작위화된 버퍼)를 리턴한다.<br>
 마지막으로, pass는 원래 buf가 replaceAllUsesWith LLVM APL을 통해 무작위화된 rnd_buf를 사용하도록 참조하는 모든 명령을 전달한다. 참고로 helper 함수 호출이 성능 오버헤드를 줄이기 위해 사용된다.<br>
 **SFR Pass**<br>
 SFR Pass는 주소를 받는 변수나 초기화되지 않은 채 사용되는 변수를 가진 모든 함수를 인식한다. 게다가, SFR pass는 스택 프레임 무작위화를 위해 VBI/SFR로 보호되는 함수의 모든 호출 장소를 계측한다.<br>
 주소를 받는 변수를 인식하기 위해, SFR 명령 모듈은 각 alloca 명령의 모든 사용을 추적하고, 함수 인수로 통과되거나 소스 피연산자로서 포인터 상술 연산과 정수 캐스팅에 관여하는 경우 주소를 받는 것으로 간주하는 모든 alloca 명령의 모든 사용을 추적하는 intra-procedure dataflow 분석을 수행한다.<br>
 초기화되지 않은 변수의 사용을 추적하기 위해, 여기서의 연구는 모든 각 stack 객체에게 intra-procedure data-flow 분석을 flow-sensitive하게, path-insensitive하게, field-sensitive한 방식으로 수행한다. 이 분석은 모든 실행 경로를 추적하고 스택 객체의 모든 사용이 적절한 초기화 지점에 의해 지배되는지 여부를 확인한다. 이렇게 함으로써, SFR pass는 적절한 초기화없이 사용될 수 있는 변수들을 결정할 수 있다.<br>
 SFR 계측에 대한 세부적인 내용을 설명하기 전에, 호출자의 stack frame의 주소를 가지는 stack pointer restore buffer(SPRB)를 설명해야 한다.<br>
 SFR은 함수로 jump 하기 전 stack frame을 무작위화한다. 따라서, SFR은 함수의 return이 호출자의 stack frame을 반드시 복구함을 보장해야 한다. 이를 위해, SaVioR 런타임 환경은 SPRB를 유지보수해야한다.<br>
 언제든 SFR로 보호되는 함수 호출이 일어날 때, 현재 stack pointer는 SPRB의 entry의 최상단에 저장되고, 상응하는 stack pointer는 함수가 return될 때 복구된다.<br>
 여기선 민감한 영역은 숨기기에 충분히 큰 64 bit 가상 주소 공간의 장점을 활용한 정보 숨기기를 통해 SPRB를 공격자로부터 숨긴다. 게다가, SaVioR는 전용 레지스터를 거치지 않는 참조가 없도록 보장한다.<br>
 이 적용에서는, SPRB 전용 레지스터로 callee 저장 레지스터이자 수기 어셈블리에서 가장 적게 사용되는 r15 레지스터를 선택했다.<br>
 위 그림의 오른쪽을 다시 보면, SaVioR이 어떻게 어셈블리 코드에서 SFR 보호 기능의 호출 장소를 계측하는 방법을 보여준다. 여기서 stack pointer rsp가 이미 slr_alloca를 통해 무작위화되었다고 가정한다.<br>
 SPRB는 완전한 내림차순 stack이므로, 전용 레지스터 r15는 유효한 주소를 포함하는 가장 높은 주소를 가리킨다. 따라서, lea 명령은 현재 stack pointer를 저장하기 전 r15를 8 byte 줄인다. stack pointer는 SPRB의 최상단에 저장된다. 그리고 stack pointer는 랜덤화된 값으로 무작위화된 stack pointer로 설정된다. 마찬가지로, stack pointer의 복구는 stack pointer의 무작위화를 제외하고 역순으로 실행된다.<br>
 SFR pass는 stack pointer와 전용 r15 레지스터에 접근하여 stack frame을 무작위화해야 한다. 이를 위해, 존재하는 존재하는 내재된 llvm, stacksave, llvm, stackrestore를 각각 사용한다.<br>
 r15 레지스터의 경우, inline 어셈블리 명령어를 삽입하여 r15 레지스터에 액세스할 수 있는 LLVM InlineAsm을 활용한다.<br>
 각각 특정 레지스터의 가져오고 설정하는 llvm.read_register와 llvm.write_register가 내재되어 있음에도 불구하고, LLVM 8.0.0은 완전히 그들을 구현하지 않는다.<br>

 **Difference between VBI and SFR passes**<br>
 공통점: 두 pass 다 LLVM IR 레벨에서 작동<br>
 차이점: SFR pass가 보호할 함수를 식별하기 위해 전체 프로그램 검사를 수행하고 해당 call-site에 SFR 계측을 적용해야 하는 반면, VBI는 그렇지 않다는 것. 따라서 VBI가 각 개별 IR 파일에서 실행되는 동안 SFR 패스는 link-time optimization(LTO)가 모든 IR 파일을 단일 IR 파일로 링크할 때까지 분석 및 계측을 수행하지 않는다.<br>
 **Callee Identification**<br>
 SFR pass는 잠재적으로 VBI or SFR 보호 기능이 대상인 모든 직간접 call-sites에 SFR 계측을 적용한다. 자세하게는, 직접 호출에 경우, SFR pass는 직접 call-sites의 대상을 명확하기 식별한다. 하지만, 간접 호출에 경우, SFR pass는 간접 call-sites의 모든 잠재적 대상을 식별하기 위해 type 기반 분석을 수행한다.<br>
 보다 구체적으로, 간접 call-sites의 인수 유형이 SFR 보호 기능 중 하나와 동일할 경우, 간접 call-sites에 SFR을 적용한다. 또한 char* 및 void*와 같은 범용 포인터 유형이 동일하다고 보수적으로 가정한다.<br>
 **Backend Modification**
 난수 생성 및 레지스터 예약을 지원하기 위해, 백엔드를 수정하였다.
 1. VBI와 SFR 패스 모두 무작위화를 위한 난수를 생성해야 한다. 이를 위해 Intel AES-NI 명령을 활용하여 무작위 숫자를 생성하는 slr_random x86 내장 함수를 구현했다.
 2. SaVioR는 계측 코드만 전용 레지스터에 접근할 수 있도록 보장한다. 컴파일된 바이너리가 전용 레지스터에 접근하는 것을 방지하기 위해, -fixed 플래그를 제공하는 GCC 컴파일러와 달리 LLVM 컴파일러는 동등한 컴파일 옵션을 지원하지 않는다. 따라서 x86_64 아키텍처의 LLVM 백엔드를 수정하여 r15, xmm14, xmm15 레지스터를 예약하였다.

 <h4>Kernel Modification</h4>
 x86_64 리눅스 커널은 사용자 프로세스에게 47 bit 가상 주소 공간을 제공한다. SaVioR은 무작위화를 위해 사용가능한 공간을 사용한다.<br>
 하지만, Linux 커널이 PIE 지원 ELF 바이너리를 ELF_ET_DYN_BASE 위에 로드되는 것을 관찰했다. 이는 SaVioR를 위한 pointer의 남은 비트가 없다는 것을 의미한다. 따라서 프로세스의 ELF segments를 더 아래로 매핑하도록 Linux 커널을 수정했다.<br>
 커널은 새로운 프로세스를 생성하는 동안 메인 실행 파일과 동적 로더를 로딩하는 역할을 한다. ELF_ET_DYN_BASE는 동적 로더와 non-PIE 주 바이너리 사이의 충분한 거리를 유지하기 위해 다이나믹 로더가 로드될 가장 낮은 주소를 지정한다. 그러나 대부분의 리눅스 배포자가 기본적으로 사전 빌드된 패키지를 PIE로 컴파일하기 때문에 ELF_ET_DYN_BASE를 0x100000000(4GB)를 낮추는 것이 합리적이라고 결론을 내린다.<br>
 또한 사용자 스택의 주소를 낮춰야 한다. 사용자 스택은 커널 로더에 의해 생성되며 STACK_TOP(=TASK_SIZE)의 음수 랜덤 오프셋으로 로드된다. 배포된 가상 스택의 수에 따라 구성 가능하며 커널 로더가 SaVioR로 보호된 애플리케이션을 초기화할 때 STACK_TOP 대신 사용된다. 전체적으로, 이러한 변경 사항들은 리눅스 커널의 19줄의 코드 차이로 구성된다.

 <h4>Runtime Library</h4>
 Linux 커널이 SaVioR로 보호된 애플리케이션의 주소 공간을 구성한 후 컴파일러-rt 런타임 지원 라이브러리는 스택 전용 영역에 여러 가상 스택을 생성하고, 전용 레지스터 r15를 SPRB를 가리키도록 설정하고, 전용 레지스터인 xmm14 및 xmm15를 초기화하여 생성된 r을 저장한다. 또한 다중 스레딩 및 non-로컬 점프를 지원한다.<br>
 런타임 지원 라이브러리는 메인 함수 이전에 실행되는 생성자 속성을 활용하는 초기화 함수를 포함한다. 이 초기화 기능은 각 스택 전용 영역에 대해 여러 스택을 생성하고 전용 레지스터 r15를 초기화한다. 좀 더 구체적으로, 이것은 pthread_atr_getstack을 사용하여 커널에 의해 만들어진 메인 스택을 찾으며 스택과 관련된 몇 가지 속성 (ex) 주소 및 길이)을 리턴한다. 그런 다음 정규 스택을 스택 전용 영역 1의 해당 스택에 미러링하고 미러링된 모든 스택이 모든 스택 전용 영역에 생성될 때까지 동일한 절차를 반복한다. 다음으로 SPRB를 가리키도록 전용 레지스터 r15가 초기화된다. 마지막으로 xmm14, xmm15 레지스터를 초기화하기 위해 런타임 라이브러리는 하드웨어 기반 엔트로피 소스에서 임의의 숫자를 반환하는 Intel의 RdRand 명령어를 사용한다.<br>
 SaVioR 지원 애플리케이션에서 멀티스레딩을 지원하기 위해 SaVioR은 다중 스택을 생성하고 새 스레드가 생성될 때 스레드 당 메타데이터(SPRB, random number와 AES 비밀 키)를 보유할 전용 레지스터를 초기화한다. 각 스레드별 메타데이터는 독립적으로 할당되고 스레드별 하드웨어 레지스터(r15, xmm14, xmm15)에 저장되므로 스레드 안전 구현에 유의하여야 한다..<br>
 구체적으로 LLVM 컴파일러는 초기화 함수(생성자)에서와 유사한 프로시저를 수행하기 위해 pthread_create 호출을 slr_pthread_create로 대체한다. 또한 스레드가 종료될 때 pthread_cleanup_push/pop을 통해 SaVioR에서 생성된 메모리를 해제한다.<br>
 마지막으로, non-로컬 점프는 Glibc의 setjmp/longjmp 구현을 수정하여 안전하게 처리된다. setjmp 함수는 현재 실행 문맥을 jmp_buf 구조에 저장한다. longjmp가 jmp_buf를 인수로 사용하여 호출되면 실행 문맥이 jmp_buf에서 복원된다. 따라서 제어 흐름은 setjmp가 호출된 시점으로 돌아간다. 원래 jmp_buf 구조는 r15 레지스터를 포함하는 callee 저장 레지스터를 연결한다. 그러나 SPRB의 기밀성과 무결성을 보장하기 위해 메모리에 푸쉬되서는 안 된다. 따라서, jmp_buf에 r15 레지스터를 저장하지 않도록 비로컬 점프 구현을 수정했다. 세부적으로, longjmp가 스택을 풀 때, SaVioR은 SPRB를 풀고 SPRB의 유효한 top entry를 가리키는 r15를 조정해야 한다. r15의 적절한 조정 없이는 SPRB는 스택 복구가 끝난 후에도 복구 stack frame을 가리키는 잘못된 항목을 가진다. 따라서, SaVioR은 현재 스택 포인터보다 낮은 주소를 가진 엔트리를 제거한다.(비교는 stack pointer의 random-bit masked 값을 사용하여 수행된다.)

 **호환 및 평가 생략**

 <h3>DISCUSSION & LIMITATIONS</h3>
 <h4>Extended virtual address space</h4>
 최근 인텔은 56 bit 사용자 가상 주소 공간을 가능하게 하는 새 5단계 페이징 모드(LA57)를 제안했다. 일부 다른 아키텍처도 48 bit 이상의 가상 주소를 지원한다. 예를 들어, 최근 SPARC 프로세서는 적어도 52bit 가상 주소를 지원하며 64bit ARM 아키텍처는 ARMv8.2-LVA 확장을 사용하는 52bit 사용자 가상 공간을 지원한다.<br>
 확장된 가상 주소 공간 덕분에 SaVioR 설계는 이러한 아키텍처와 상당히 호환되어 x86_64로 제한되지 않는다.
 <h4>Entropy of ASLR</h4>
 현대 os에 걸친 SaVioR의 호환성을 보여주기 위해 OpenBSD, HardenedBSD, 윈도우(x86_64)에서 ASLR 구현 메뉴의 엔트로피를 제시한다. OpenBSD 및 HardenedBSD에서의 ASLR 구현을 수동으로 분석하였다.<br>
 전에 보았듯 리눅스는 각각 28, 13, 28, 22 bit의 엔트로피를 제공한다.

 **OpenBSD and HardenedBSD**<br>
 OpenBSD는 ELF,mmap/malloc, stack segments에 각각 32, 20, 6 bit의 엔트로피틑 제공하여 Linux보다 낮은 엔트로피를 제공한다. 즉 더 많은 무작위화 엔트로피를 허용한다. 반대로 HardenedBSD는 30, 30, 33bit의 ASLR 엔트로피를 제공하는 최신 보안 기능을 채택하여 5단계 페이징을 통해 1024개의 스택만을 배포할 수 있다.<br>
 **Windows**<br>
 Windows는 기본적으로 17,19,8,17bit의 엔트로피를 제공하므로 사용 가능한 64bit 가상 공간을 완전히 활용하지 못한다. 이 문제를 해결하기 위해 High Entropy ASLR(HEASLR)이 도입되어 각 세그먼트에 17,19,24,33비트의 엔트로피를 제공한다.<br>
 흥미롭게도 스택을 제외하고는 리눅스가 windows보다 ASLR 엔트로피를 더 많이 제공함을 볼 수 있다. HEASLR 지원 애플리케이션에서 SaVioR은 46bit 가상 주소 공간도 예약해야 하며 5단계 페이징을 통해 1024개의 스택을 구축할 수 있으므로 stack layout에 상당한 수준의 불확실성을 유도한다.
 <h4>Intel Control-flow Enforcement Technology</h4>
 Intel CET(Control-flow Enforcement Technology)는 전방 및 후방 control-flow의 무결성을 보호하기 위한 두 가지 기술을 제공한다.

 1. 간접 함수 호출이 제어 흐름을 함수 진입점에만 전송할 수 있도록 보장하는 거친 전방 및 후방 control-flow 정책
 2. 하드웨어 정책의 shadow stack을 이용한 역방향 간선 보호<br>
 <br>

 자세히는 Intel CET는 순방향 간선 보호를 위해 함수 진입 지점에 위치하고 유효한 분기 대상을 표시하는 새 명령어인 ENDBRANCH를 도입했다. 역방향 간선 보호를 위해, 하드웨어로 격리된 shadow stack에서 return 주소를 저장하고 가져오기 위해 call/ret 명령의 의미를 변경한다.<br>
 그러나 순방향 간선 보호는 거친 정책 때문에 최근의 CFI 우회 공격에 아직 취약하다.<br>
 이를 극복하기 위해, Intel CET는 SaVioR와 결합하여 스택 상의 함수 포인터의 위치를 무작위화해 함수 포인터 손상 공격에 대한 허들을 높일 수 있다.<br>
 역방향 간선 보호는 반환 주소의 무결성만 보장하며 스택의 비제어 및 기타 제어 정보(함수 포인터)를 보장하지 않는다. 더더욱, 그것은 call/ret 명령의 의미를 바꾸기 때문에, return 주소의 보호에 맞춰져있다. 따라서, 다른 데이터를 보호하기 위해 이 기법을 다른 데이터를 보호하기 위해 기법을 용도 변경하는 것은 쉽지 않다.<br>
 이러한 제한은 SaVioR을 채택함으로써 극복할 수 있다.

 <h3>CONCLUSION</h3>
 스택의 레이아웃에 예측 불확실성을 도입하고 C/C++ 애플리케이션에 대한 스택 기반 공간적/시간적 메모리 수정 공격으로부터 포괄적 보호를 제공하는 SaVioR를 제시하였다. 이를 위해 64bit 가상 주소 공간과 pointer mirroring을 활용, 각 함수 호출 시 stack 상주 객체의 위치를 효율적으로 무작위화한다.<br>
 통계 및 경험적 보안 평가는 SaVioR이 높은 확률로 공간적/시간적 공격을 효과적으로 완화한다는 것을 검증했다.<br>
 또한 다양한 아키텍처와 운영 체제에서 채택될 수 있을 만큼 충분히 실현 가능하고 실용적이라는 것을 보여주었다.

 <h2>작성자의 요약</h2>
 SaVioR는 스택을 통한 공간적/시간적 메모리 오염 공격을 방어하기 위해 종합적인 메모리 보호 기법을 설명한다.<br>
 SaVioR는 두 가지 무작위화 방법을 제시한다. **VBI와 SFR**이다.<br>
![image](https://user-images.githubusercontent.com/73513005/148339074-a16d0a18-0a9c-4437-9cf5-29a3b232b0cd.png)<br>
 VBI는 공격 가능한 버퍼에서 오버플로우 같은 공간적 공격을 방지하기 위해, 취약한 변수들을 목표 변수로부터 분리, 격리시킨다. 위 그림과 같이 오버플로우가 발생해도, 다른 스택에 있는 목표 변수들은 안전한 것을 확인할 수 있다.<br>
 그러나 취약한 변수들만을 격리시킨다면, 어차피 한 줄에 목표 변수들이 존재할 것이므로 이는 다른 함수의 목표 변수들을 건드릴 수 있다는 뜻이다. 이를 막기 위해 SFR이 도입된다.<br>
 ![image](https://user-images.githubusercontent.com/73513005/148339483-f3fea1f7-cb0c-4a9d-acfc-80f18d2d9158.png)<br>
 SFR은 함수에 padding을 넣고, 각 함수들이 위치한 스택을 무작위화함으로써 공간적 공격은 물론 시간적 공격도 방지할 수 있다.<br>
 위 그림을 보면 기존 애플리케이션에서는 UR이 포함된 func3의 void* func_ptr으로 인해 공격자가 그 안에 값을 미리 넣어놓음으로써 func2의 공격자가 조작 가능한 메모리로 유도할 수 있음을 보여주고 있다.<br>
 이에 SFR은 메모리를 무작위 스택에 배치하고, padding을 넣어 조작 위치의 불확실성을 더함으로써 공격자가 UR에 원하는 값을 집어넣기 상당히 어려워짐을 보여주고 있다.<br>
 SFR은 상당한 성능 오버헤드를 발생시키고, 이를 방지하기 위해 목표 가능성이 담긴 변수들을 담는 Stack frame을 식별하여 그 stack frame의 위치만 무작위화 할 것이다.<br><br>
 ---<br>
 SaVioR의 가상 주소 공간의 구조는 기존 영역과 stack-only 영역으로 나뉜다.<br>
 여기서 무작위화를 위하여, pointer에서 가상 주소 공간으로 지정되는 bit를 희생하여 random bit를 저장한다. 이 random bit가 함수가 위치할 가상 주소 stack의 위치가 된다. 이 random bit가 변경되는 것을 pointer mirroring이라고 부른다.<br><br>
 ---<br>
 SaVioR는 스택에 추가 무작위성을 도입하기에, ASLR과 완벽히 호환되어야 하며 이는 ASLR의 엔트로피를 훼손해서는 안된다는 의미이다.<br>
 여기서 리눅스는 사용가능한 bit로 각각 40, 25,40, 32 bit의 주소 지정가능 공간이 필요하며, 이는 오직 5개의 random bit를 지원하여 부족할 수 있지만, 여기선 인텔이 제시한 5단계 페이징을 통해 주소지정가능 공간을 57bit까지 확장해 14 random bit를 통해 최대 16,384개의 스택을 지원가능하게 한다.<br><br>
 ---<br>
 SaVioR를 지원하기 위해, 난수를 생성하는 slr_random 함수와 스택 위치를 무작위화 해주는 slr_alloca 명령을 LLVM에 추가했다. 여기서 난수 생성은 전용 레지스터인 xmm14, xmm15에 저장된다.<br>
 VBI pass는 버퍼를 위에 언급된 slr_random, slr_alloca를 통해 스택 내의 위치를 바꾸고 바꾼 위치를 돌려 받는다. 이를 다른 호출 함수가 부를 수 있도록 수정하는 과정을 prologue에서 수행한다.<br>
 SFR pass는 모든 각 stack 객체에게 intra-procedure data-flow 분석을 flow-sensitive하게, path-insensitive하게, field-sensitive한 방식으로 수행한다. 이 분석은 모든 실행 경로를 추적하고 스택 객체의 모든 사용이 적절한 초기화 지점에 의해 지배되는지 여부를 확인한다. 이렇게 함으로써, SFR pass는 적절한 초기화없이 사용될 수 있는 변수들을 결정할 수 있다.<br>
 또한 함수 리턴이 반드시 호출자에 stack frame으로 돌아가기 위해, stack pointer restore buffer(SPRB)에 호출자의 주소를 보관한다.<br>
 언제든 SFR로 보호되는 함수가 호출되면, 현재의 스택 포인터가 SPRB의 entry의 최상단에 저장되고, return 될 때 복구된다. 이 SPRB는 잘 사용되지 않는 r15 레지스터에 저장된다.<br>
 실제 실행에서는, r15를 8바이트 줄인 다음 stack pointer가 들어가게 되고, stack pointer가 무작위값으로 설정된 새 stack pointer로 설정된다. return은 이와 역순으로 실행되게 된다.<br><br>
 ---<br>
 두 VBI, SFR은 둘 다 LLVM IR에서 작동한다.<br>
 그러나 SFR은 보호할 함수를 구분하기 위해 전체 프로그램 검사를 수행해야 하고 식별된 대상만 적용해야 하는 반면 VBI는 그렇지 않다.<br><br>
 ---<br>
 또한 구현을 위해 난수를 위한 Intel AES-NI 명령을 활용하여 slr_random 명령어를, 전용 레지스터에 계측 코드만 접근할 수 있도록 LLVM backend를 수정해 r15, xmm14, xmm15를 예약하였다.<br>
 커널 수정 부분은 지식이 부족하여 완벽히 정리하진 못했지만, 현재 47bit 가상 주소 공간만 주어져 비트가 부족한 현상이 발생하여, ELF segments를 더 아래로 매핑하도록 수정했다. 후략<br><br>
 ---<br>
 다른 os에서 ASLR의 엔트로피를 해치지 않으며 SaVioR의 핵심인 random bit를 비교해 본 결과, openBSD는 높은 무작위 엔트로피를 얻을 수 있었지만 HardenedBSD는 5단계 페이징을 사용해도 1024개의 스택을 얻을 수 있었다. windows는 HEASLR을 도입하는데, 5단계 페이징을 통해 마찬가지로 1024개의 스택을 구축할 수 있어 상당한 불확실성을 유도할 수 있다.<br><br>
 ---<br>
 Intel CET는 최신 CFI 우회 공격에 취약한데, SaVioR와 결합해 함수 포인터 손상 공격에 대한 허들을 높일 수 있고 반환 주소의 무결성만 보장하던 방식을 다른 데이터를 보호하는데 SaVioR를 채택해 극복할 수 있다.