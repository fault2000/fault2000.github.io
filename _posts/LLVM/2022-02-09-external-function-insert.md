---
layout: post
title: LLVM 외부 함수 호출
category: [llvm, security]
tags: [llvm, pass, optimization]
fullview: true
comments: true
author: fault2000
---

## 서론

이번 목표는 API를 호출하는 pass를 구현하는 것으로, 교수님이 들어주신 예는 1) 예제 함수를 .so 형태로 컴파일 후, 2) llvm pass를 통해 임의의 위치에 .so 파일 내의 특정 함수 호출을 삽입 이라는 두 가지 과정이라고 간략하게 설명해주셨다.  
그러나 잠시 생각한 후 문제에 봉착했는데, 이게 간단한 컴파일 과정이라면 위 방법은 분명 가능하지만, pass가 단순한 컴파일이 아닌 llvm pass manager를 통해 사용되고, 이 과정에서 pass가 어떤 라이브러리를 사용할지 지정해주는 옵션이 부재되어 있다.  
이러한 점 때문에 다른 접근 방식을 생각해보게 되었고, 기존 함수 자체에 존재하는 함수는 사용할 수 있다는 점(자세한 부분은 [이 링크](https://stackoverflow.com/questions/51082081/llvm-pass-to-insert-an-external-function-call-to-llvm-bitcode)를 참고)을 사용하는 법을 고안했다.  
아니, 외부에 있는 함수를 넣어야 하는데 위 링크의 방식은 이미 모듈 내의 존재하는 print 함수를 받아서 필요할 때마다 넣어주는 방식이 아니냐, 전혀 도움이 안 된다...... 고 생각하실 수 있지만, 약간의 응용을 생각해냈는데, [이 링크](https://sites.google.com/site/arnamoyswebsite/Welcome/updates-news/llvmpasstoinsertexternalfunctioncalltothebitcode)에서 제안하는 방식을 보면, llvm-link를 이용해 두 비트코드 파일인 bc를 합쳐주는 방식에서 영감을 받아, 우리가 삽입하기를 원하는 함수와 우리가 opt 해줄려는 함수를 합쳐준다. 이후 pass를 사용하면, 우리가 원하는 함수를 삽입할 수 있을 것이다.  
이러한 방식은 약간 원하던 방식과 조금 동떨어져있긴 한데, 일단 작동하는지 확인해보자.
***
먼저 테스트할 코드를 간단히 작성해보았다.

### test1.c

```c++
#include <stdio.h>

void api(int a) {
    printf("Work!\n");
}
```

### test2.c

```c++
#include <stdio.h>

int main() {
    int a = 1;
    int b = 2;
    if (a==1) {
        printf("output = %d", a+b);
    }
}
```

위 두 코드가 사용될 테스트 코드이다. 구조는 상당히 단순하므로 설명은 패스하고, test1.c가 우리가 삽입하려는 코드이다. 여기서 주의할 점은 저 api라는 이름이 test2.c에 동시에 존재하면 문제가 발생하므로, 이름을 좀더 복잡하게 만들어주거나 test2.c의 겹치는 함수 이름을 수정해주어야할 필요가 있다.
이제 이들을 .bc 확장자로 만들어준다.

![image](https://user-images.githubusercontent.com/73513005/153349955-05d99780-5d88-49dc-85ad-3f1778d280f9.png)

결과물은 아래와 같다. 보시다시피 안에 api와 printf, main 같은 함수가 모두 있음을 볼 수 있다.

<img width="714" alt="image" src="https://user-images.githubusercontent.com/73513005/153350032-2993781e-4e9f-42aa-b886-77d67212c5ad.png">

이제 우리가 사용할 pass는 다음과 같다.

```c++
> cat ../llvm/lib/Transforms/Api/Api.cpp
#include "llvm/Pass.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/IR/Type.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/Support/raw_ostream.h"

#include "llvm/IR/IRBuilder.h"

#include <vector>

using namespace llvm;

namespace{
struct api : public ModulePass{
    static char ID;
    Function *monitor;

    api() : ModulePass(ID) {}

    virtual bool runOnModule(Module &M) override
    {
        int counter = 0;

        for(Module::iterator F = M.begin(), E = M.end(); F!= E; ++F)
        {
            errs() << "Function name: " << F->getName() << "\n";
                if(F->getName() == "api"){
                    monitor = cast<Function>(F);
                    continue;
                }

                for(Function::iterator BB = F->begin(), E = F->end(); BB != E; ++BB)
                {
                    for(BasicBlock::iterator BI = BB->begin(), BE = BB->end(); BI != BE; ++BI)
                    {
                        if(isa<BranchInst>(&(*BI)))
                        {
                            errs() << "Found Branch!" << '\n';
                            ArrayRef< Value* > arguments(ConstantInt::get(Type::getInt32Ty(M.getContext()), counter, true));
                            counter++;
                            Instruction *newInst = CallInst::Create(monitor, arguments, "");
                            BB->getInstList().insert(BI, newInst);
                        }
                    }
                }
        }
        return true;
    }
};
char api::ID = 0;
static RegisterPass<api> X("api", "LLVM api Pass");

}
```

위 코드가 사용할 패스로, 간략하게 설명하자면 모듈을 돌면서 api라는 이름의 함수를 찾고, 찾은 함수를 branch 명령이 일어나는 BasicBlock 뒤에 우리가 삽입하고자 하는 api 함수를 넣어주는 동작을 한다. 여기서 api라는 함수의 이름이 겹치면 안되고, 또한 우리가 삽입하기를 원하는 함수가 맨 앞에 오도록 조정해주어야할 필요가 있다.  
이제 이 패스를 적용해보자.  

<img width="713" alt="image" src="https://user-images.githubusercontent.com/73513005/153351145-6e3ba979-d0c5-403b-9149-2893b5a3cdd1.png">

함수 이름을 출력해줌으로써 혹시라도 겹치는 함수 이름이 있는지 체크할 수 있는 편의성을 생각했고, 함수가 들어가는지 확인할 수 있는 기능도 있음을 알 수 있다. 사진에서는 Found Call이라고 적혀있지만 오타는 애교로 넘어가주자. 이제 결과로 출력된 바뀐 output.bc를 확인해보자.  

<img width="168" alt="image" src="https://user-images.githubusercontent.com/73513005/153351452-08d50f58-148b-47a9-b22b-b4a6b729f514.png">

우리가 원하는 함수가 잘 입력되었음을 확인할 수 있다. 이제 여기서 조건을 바꿈으로써 원하는 삽입 위치를 조절할 수 있고, test1.c를 변경함으로써 원하는 삽입 함수를 변경해줄 수 있다.  
물론 단점도 존재한다. 테스트할 .bc 파일이 많으면 많을수록 link 과정이 많아지고, 기존 목표는 .so 공유 라이브러리를 활용하는 것이지만 조금 동떨어진 방법을 사용하였다. 하지만 결국 목표를 완수할 수 있음을 보여줄 수 있었고, 이번 시연에서 인자를 활용하지는 않았지만 코드를 자세히 보면 인자를 넘겨줄 수 있음도 볼 수 있다. 이를 통해 더 다양한 활용도 가능하므로 어느 정도 만족스러운 결과를 얻은 것 같다.
