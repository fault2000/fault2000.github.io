---
layout: post
title: LLVM pass 예시
category: [llvm]
tags: [llvm, pass, optimization, tutorial]
fullview: true
comments: true
author: fault2000
---

각 패스들의 종류들을 알아봤으니, 이제 여러가지 예시 패스들을 만들어보고, 작동 방식을 살펴보도록 하겠다.  

<img width="836" alt="스크린샷 2022-01-27 오전 3 09 29" src="https://user-images.githubusercontent.com/73513005/151221781-f7e484b3-1c30-473b-bb6f-2d90059aa1fe.png">

위 사진은 functionPass를 활용하여 각 함수에 들어 있는 명령들의 종류와 각 호출 횟수를 알려주는 패스이다. 일단 결과는 아래와 같이 나온다.  

<img width="705" alt="스크린샷 2022-01-27 오전 3 24 41" src="https://user-images.githubusercontent.com/73513005/151223805-246e2469-8fae-44f4-832a-29e62479d385.png">

이제 코드를 살펴보는 시간을 가지겠다.  

```c++
#define DEBUG_TYPE "opCounter"
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/raw_ostream.h"
#include <map>

using namespace llvm;
namespace {
	struct CountOp : public FunctionPass {
		std::map<std::string, int> opCounter;
		static char ID;
		CountOp() : FunctionPass(ID) {}
		virtual bool runOnFunction(Function &F) {
			errs() << "Function " << F.getName() << '\n';
			for (Function::iterator bb = F.begin(), e = F.end(); bb != e; ++bb) {
				for(BasicBlock::iterator i = bb->begin(), e = bb->end(); i != e; ++i) {
					if(opCounter.find(i->getOpcodeName()) == opCounter.end()) {
						opCounter[i->getOpcodeName()] = 1;
					} else {
						opCounter[i->getOpcodeName()] += 1;
					}
				}
			}
			std::map <std::string, int>::iterator i = opCounter.begin();
			std::map <std::string, int>::iterator e = opCounter.end();
			while (i != e) {
				errs() << i->first << ": " << i->second << "\n";
				i++;
			}
			errs() << "\n";
			opCounter.clear();
			return false;
		}
	};
}
char CountOp::ID = 0;
static RegisterPass<CountOp> x("opCounter", "Counta opcodes per functions");
```

전체 코드는 위와 같고, 이 중 중요한 부분만 보도록 하자.

```c++
virtual bool runOnFunction(Function &F) {
    errs() << "Function " << F.getName() << '\n';
    for (Function::iterator bb = F.begin(), e = F.end(); bb != e; ++bb) {
        for(BasicBlock::iterator i = bb->begin(), e = bb->end(); i != e; ++i) {
            if(opCounter.find(i->getOpcodeName()) == opCounter.end()) {
                opCounter[i->getOpcodeName()] = 1;
            } else {
                opCounter[i->getOpcodeName()] += 1;
            }
        }
    }
    std::map <std::string, int>::iterator i = opCounter.begin();
    std::map <std::string, int>::iterator e = opCounter.end();
    while (i != e) {
        errs() << i->first << ": " << i->second << "\n";
        i++;
    }
    errs() << "\n";
    opCounter.clear();
    return false;
}
```

가장 중요한 부분은 단연코 직접 구현할 내용 부분이 되겠다. 먼저 함수의 이름을 출력하는 줄을 간단한 getName으로 만들어준다.  
이제 for문 2 중첩이 나오는데, 여기서 Function::iterator를 통해 각 함수 속 basicBlock을, 한 번 더 들어가서 basicBlock 속 instruction을 iterator를 통해 내용을 체크한다. 체크하는 방법은 map 자료구조를 가진 opCounter에 데이터를 저장하여 내용물이 없다면 새로 추가하고, 있다면 그 내용물 속 값을 더해주는 식으로 구현되었다.  
마지막으로 opCounter의 내용을 iterator를 사용해 순회하면서 출력해준다. 그 뒤 내용을 지우고 종료한다.  
이제 한번 실습을 위해 ModulePass로 변환해 보았다. 결과는 아래와 같다.

```c++
#define DEBUG_TYPE "opCounter"
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Module.h"
#include "llvm/Support/raw_ostream.h"
#include <map>

using namespace llvm;
namespace {
	struct CountOp : public ModulePass {
		std::map<std::string, int> opCounter;
		static char ID;
		CountOp() : ModulePass(ID) {}
		virtual bool runOnModule(Module &M) {
			for (Module::iterator F = M.begin(), e = M.end(); F != e; ++F) {
				for (Function::iterator bb = F->begin(), e = F->end(); bb != e; ++bb) {
					for(BasicBlock::iterator i = bb->begin(), e = bb->end(); i != e; ++i) {
						if(opCounter.find(i->getOpcodeName()) == opCounter.end()) {
							opCounter[i->getOpcodeName()] = 1;
						} else {
							opCounter[i->getOpcodeName()] += 1;
						}
					}
				}
			}
			std::map <std::string, int>::iterator i = opCounter.begin();
			std::map <std::string, int>::iterator e = opCounter.end();
			while (i != e) {
				errs() << i->first << ": " << i->second << "\n";
				i++;
			}
			errs() << "\n";
			opCounter.clear();
			return false;
		}
	};
}
char CountOp::ID = 0;
static RegisterPass<CountOp> x("opCounter", "Counta opcodes per functions");
```

보면 Module이 전체 프로그램이고, 그 안에 여러 함수가 있으며, 또 그 안에는 여러 basicblock이, 그 안에는 여러 instruction이 있는 형태이므로 우리의 코드를 약간만 변형시켜주면 같은 기능을 하는 ModulePass를 만들어 줄 수 있다.  
LLVM의 자료구조는 다음과 같다.

<img width="540" alt="스크린샷 2022-01-27 오후 3 12 25" src="https://user-images.githubusercontent.com/73513005/151302027-6fab3c7e-a6d4-49d1-be46-c175ad0f5c37.png">

돌려본 결과는 다음과 같다.

<img width="703" alt="스크린샷 2022-01-27 오후 3 08 00" src="https://user-images.githubusercontent.com/73513005/151301569-139f78ad-1c50-4489-9fc3-23d2c0f1285b.png">

