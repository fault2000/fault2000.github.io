---
layout: post
title: Memory tagging extension, MTE tutorial with docker on window
category: [intern, security]
tags: [mte, arm, memory safety]
fullview: true
comments: true
author: fault2000
use_math: true
---

인턴 활동 중 armv8.5-a 버전에서 추가된 기능인 memory tagging extension(mte)에 대한 기능을 살펴보고, 직접 기능을 실습해보았다. 그 뻘짓에 대한 기록을 남긴다.  
이 포스트의 모든 과정은 window 11과 10에서 진행되었다. window 11에서 도커 설치 중 오류가 있으니 참고바란다.
이 글은 mte에 대한 지식을 미리 가지고 실습을 진행하고 싶은 사람을 대상으로 씀을 밝힌다.  

## 서론

mte를 직접 실습하기 위해, qemu를 먼저 들여다보았으나 docker를 활용하면 편하겠다는 생각에 둘 다 진행해보았다. 먼저 비교적 간단한 도커를 활용한 방법을, 그 다음 포스트에 qemu를 통해 진행한 과정을 정리했다.  

Docker는 간단하다. 이미 존재하는 이미지를 가져와 그 안에서 mte 관련 코드를 실행만 하면 수행될 수 있다.  
docker를 다루는 법을 설명하자니 너무 방대해질 것 같아 그 부분은 미리 공부하고 오길 권한다. 물론 여기서 자세히 과정을 설명할 것이기에, 그냥 따라오기만 해도 된다. visual studio의 확장 프로그램으로 쓰면 상대적으로 편하게 쓸 수 있으니 그 과정을 기술하겠다.  

<img width="1280" alt="image" src="https://user-images.githubusercontent.com/73513005/163657449-dfe7a815-9248-44c4-b9c3-889222087bad.png">  

먼저 비주얼 스튜디오 확장에서, docker를 검색하면 제일 먼저 공식 도커를 제일 위에서 발견할 수 있다. 이를 받으면 왼쪽 라인에 도커 항목이 생김을 확인할 수 있고, 도커를 다운로드하려는 창이 나오게 된다. 여기서 설치를 선택해도 되고,  

<img width="1280" alt="image" src="https://user-images.githubusercontent.com/73513005/163657609-a1aa23e4-5aa9-4c8f-a198-3fbe6a9c6b63.png">  

위 그림처럼 도커 항목에 들어가 왼쪽 아래의 help and feedback 항목에 설치를 지원하고 있으니, 참고하면 좋다.  
다운로드가 전부 끝났다면 진행할 수 있으며, 그래도 활성화가 안 된다면 설치 후 재부팅을 한 번할 것을 권한다.

이 포스트에서는 직접 만들어 놓은 이미지가 있으니 이것을 쓰도록 하자.

## 빌드된 이미지 활용

먼저 도커가 설치된 비주얼 스튜디오에서 터미널을 열고 다음과 같은 명령어를 기입한다.

```terminal
docker pull fault2000/ubuntu_mte_test
```

arm 우분투에 우리가 필요한 gcc만 받아놓은 이미지이다. 크게 변경된 것도 없다.

<img width="1280" alt="image" src="https://user-images.githubusercontent.com/73513005/163659496-4d2aac6d-457d-4dcf-a591-36350f8203a6.png">  

다 받으면 왼편 이미지란에 새로운 이미지가 들어와 있음을 확인할 수 있을 것이다.  

<img width="1280" alt="image" src="https://user-images.githubusercontent.com/73513005/163659563-4f2e2b73-c640-48ad-b529-6144b4dcf4c6.png">  

이제 이미지를 컨테이너에 옮길 차례이다. 이미지를 눌러 활성화해주고, 아래 latest를 우클릭하여 나온 run interactive를 눌러주자.  

<img width="1280" alt="image" src="https://user-images.githubusercontent.com/73513005/163660149-47bc2765-34a9-4f22-b4d3-9991dcfe5abf.png">

왼편 컨테이너에 새로운 컨테이너가 추가되고, 터미널에는 새로운 터미널이 뜬다. ls를 눌러보니 test.c와 test가 보인다. 이는 [Memory Tagging Extension (MTE) in AArch64 Linux](https://www.kernel.org/doc/html/latest/arm64/memory-tagging-extension.html) 저널의 예시를 가져온 것이나, 수정된 부분이 한 가지 있다. 코드 전문은 아래와 같다.  

```c++
/*
 * -march=armv8.5-a+memtag을 gcc에 flag로 주어야 한다.
 */
#include <errno.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/auxv.h>
#include <sys/mman.h>
#include <sys/prctl.h>

/*
 * From arch/arm64/include/uapi/asm/hwcap.h
 */
#define HWCAP2_MTE              (1 << 18)

/*
 * From arch/arm64/include/uapi/asm/mman.h
 */
#define PROT_MTE                 0x20

/*
 * From include/uapi/linux/prctl.h
 */
#define PR_SET_TAGGED_ADDR_CTRL 55
#define PR_GET_TAGGED_ADDR_CTRL 56
# define PR_TAGGED_ADDR_ENABLE  (1UL << 0)
# define PR_MTE_TCF_SHIFT       1
# define PR_MTE_TCF_NONE        (0UL << PR_MTE_TCF_SHIFT)
# define PR_MTE_TCF_SYNC        (1UL << PR_MTE_TCF_SHIFT)
# define PR_MTE_TCF_ASYNC       (2UL << PR_MTE_TCF_SHIFT)
# define PR_MTE_TCF_MASK        (3UL << PR_MTE_TCF_SHIFT)
# define PR_MTE_TAG_SHIFT       3
# define PR_MTE_TAG_MASK        (0xffffUL << PR_MTE_TAG_SHIFT)

/*
 * Insert a random logical tag into the given pointer.
 */
#define insert_random_tag(ptr) ({                       \
        uint64_t __val;                                 \
        asm("irg %0, %1" : "=r" (__val) : "r" (ptr));   \
        __val;                                          \
})

/*
 * Set the allocation tag on the destination address.
 */
#define set_tag(tagged_addr) do {                                      \
        asm volatile("stg %0, [%0]" : : "r" (tagged_addr) : "memory"); \
} while (0)

int main()
{
        unsigned char *a;
        unsigned long page_sz = sysconf(_SC_PAGESIZE);
        unsigned long hwcap2 = getauxval(AT_HWCAP2);

        /* MTE가 존재하는지 체크 */
        if (!(hwcap2 & HWCAP2_MTE))
                return EXIT_FAILURE;

        /*
         * Enable the tagged address ABI, synchronous MTE 
         * tag check faults (based on per-CPU preference) and allow all
         * non-zero tags in the randomly generated set.
         * tagged address ABI, synchronous MTE tag check를
         * 사용하도록 설정, 임의로 생성된 집합에서 0이 아닌 태그 허용
         */
        if (prctl(PR_SET_TAGGED_ADDR_CTRL,
                  PR_TAGGED_ADDR_ENABLE | PR_MTE_TCF_SYNC |
                  (0xfffe << PR_MTE_TAG_SHIFT),
                  0, 0, 0)) {
                perror("prctl() failed");
                return EXIT_FAILURE;
        }

        a = mmap(0, page_sz, PROT_READ | PROT_WRITE,
                 MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
        if (a == MAP_FAILED) {
                perror("mmap() failed");
                return EXIT_FAILURE;
        }

        /*
         * Enable MTE on the above anonymous mmap. The flag could be passed
         * directly to mmap() and skip this step.
         * 위의 익명의 mmap에 MTE를 활성화한다.
         * mmap에 직접 flag를 줌으로써 이 과정 건너뛰기 가능
         */
        if (mprotect(a, page_sz, PROT_READ | PROT_WRITE | PROT_MTE)) {
                perror("mprotect() failed");
                return EXIT_FAILURE;
        }

        /* default tag (0)으로 접근 */
        a[0] = 1;
        a[1] = 2;

        printf("a[0] = %hhu a[1] = %hhu\n", a[0], a[1]);

        /* set the logical and allocation tags
        * 논리적 태그 및 할당 태그 설정 */
        a = (unsigned char *)insert_random_tag(a);
        set_tag(a);

        printf("%p\n", a);

        /* non-zero tag access
        * 0이 아닌 태그 접근 */
        a[0] = 3;
        printf("a[0] = %hhu a[1] = %hhu\n", a[0], a[1]);

        /*
         * If MTE is enabled correctly the next instruction will generate an
         * exception.
         *만약 MTE가 정확히 활성화되어있다면 다음 명령은 오류를 발생시킬 것이다.
         */
        printf("Expecting SIGSEGV...\n");
        a[16] = 0xdd;

        /* this should not be printed in the PR_MTE_TCF_SYNC mode
        * 다음은 PR_MTE_TCF_SYNC 모드에서는 출력되지 않아야 한다. */
        printf("...haven't got one\n");

        return EXIT_FAILURE;
}
```

처음 실습 진행 시 저널에 코드를 그대로 가져왔으나, 무언가 잘못되어 맨땅에 해딩을 엄청나게 박았다.  
위 코드와 저널의 코드 자체는 거의 완벽하게 같으나 prctl 함수에 수정이 가해졌다.  
저널 본문에서 main에 prctl 함수는 다음과 같이 나와있다.  

```c
if (prctl(PR_SET_TAGGED_ADDR_CTRL,
                  PR_TAGGED_ADDR_ENABLE | PR_MTE_TCF_SYNC | PR_MTE_TCF_ASYNC |
                  (0xfffe << PR_MTE_TAG_SHIFT),
                  0, 0, 0)) {
                perror("prctl() failed");
                return EXIT_FAILURE;
        }
```

우리의 코드에서는 prctl의 인자 중 PR_MTE_TCF_ASYNC가 사라졌다. 저널에서는 sync와 async를 동시에 활성화하면 per-cpu preferred tag checking mode가 활성화된다고 나와있지만, 우리의 환경에서는 둘 다 활성화시킬 시 prctl에서 Invalid argument 오류가 발생한다. 아쉽지만 우리는 PR_MTE_TCF_SYNC만 활성화해주자. 사실 이 실습에서는 크게 문제될 것 없는 요소이기도 하다.  
이미 gcc로 컴파일된 친구가 test이기에, 실행시켜보자.  

<img width="377" alt="image" src="https://user-images.githubusercontent.com/73513005/163660522-50a7e67e-2110-4c61-854c-31cedcd781a7.png">

의도한 대로 잘 실행됨을 볼 수 있다. 코드가 어떻게 굴러가는지는 위 코드 전문의 주석을 참고하면 좋다.

## 결론

이 포스트에서 대충 docker를 이용해 수동으로 mte를 설정하고, 실행시켜보았다. 다음 포스트에서는 리눅스에서 qemu를 통해 mte를 실행하는 과정을 보이도록 하겠다.