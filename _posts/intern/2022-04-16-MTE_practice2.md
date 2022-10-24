---
layout: post
title: Memory tagging extension, MTE tutorial with qemu
categories: [intern]
tags: [mte, arm, memory safety]
fullview: true
comments: true
author: fault2000
---

## 서론

이번 포스트에서는 qemu를 이용해 mte를 실습해보는 과정을 담았다.  
qemu가 리눅스에 초점을 맞추었으므로, 이번 실습은 가상 머신을 사용해 ubuntu 환경을 조성하여 거기서 qemu 에뮬레이션을 돌리는 방법을 채택했다.  
먼저 아무 가상 머신을 사용하여 우분투를 설치하자. 필자는 vmware에 우분투 21.10 버전을 설치하였다.  

먼저 우분투를 처음 설치하면 필요한 가장 필수 파트인 update, upgrade 과정을 수행하자.

```terminal
sudo apt update
sudo apt upgrade
```

## 이미지

그 다음 가장 중요한 이미지이다.  
[What’s new in security for Ubuntu 21.04?](https://ubuntu.com/blog/whats-new-in-security-for-ubuntu-21-04)에서 밝히다시피 21.04에서 arm64 memory tagging을 지원함을 밝히고 있다. 즉 우리는 우분투 21.04 버전 이상을 사용하여야 한다.  
우리는 이미 만들어진 클라우드 img 파일을 사용하여 불필요한 설치 과정을 생략하지만, 따로 용량을 늘리는 작업을 병행해주어야한다. 일단 이미지 파일은 아래에서 받아주자.  
[https://cloud-images.ubuntu.com/releases/21.10/release-20220309/](https://cloud-images.ubuntu.com/releases/21.10/release-20220309/)  
원래는 클라우드 서버에 활용되기 위해 이미 설치된 img들이지만, 우리의 용도에 맞으므로 그냥 사용하였다.  
ubuntu-20.04-server-cloudimg-arm64.img 파일을 골라 다운받아주면 된다. 잘 안보인다면 그냥 Cloud image칸에 64-bit ARM Cloud image를 선택해주어도 바로 다운로드가 시작된다.  

## 필요 도구 설치

제목에서도 알 수 있다시피, qemu를 이용해 진행할 것이므로 qemu가 필요하다. 다음 명령어를 입력하자.

```terminal
sudo apt-get install -y qemu-utils qemu-system-arm
```

또한 우리의 과정에서 flash image들이 필요하므로 다음과 같은 명령어도 필요하다.  

```terminal
dd if=/dev/zero of=flash1.img bs=1M count=64
dd if=/dev/zero of=flash0.img bs=1M count=64
dd if=/usr/share/qemu-efi-aarch64/QEMU_EFI.fd of=flash0.img conv=notrunc
```

이들은 추후 qemu에 인자로 넘겨질 이미지들이다.

## 이미지 용량 증가

위 이미지를 바로 사용하기에는 용량이 필요한 만큼만 있어 우리 작업을 하기엔 부족하다. 즉, 용량 증가 과정을 따로 거쳐주어야 한다.
우리가 위에서 설치한 qemu-utils의 qemu-img를 통해 img의 크기를 늘려줄 것이다. 다음 명령어를 통해 크기를 조절하자. 여기서는 20G정도를 할당했지만, 우리가 할 작업은 큰 용량을 필요로 하지 않으므로 10G나 사양에 맞추어 더 줄여도 문제 없을 것이다.

```terminal
qemu-img resize ubuntu-21.10-server-cloudimg-arm64.img 20G
```

조절이 완료되었다면 Image resized가 출력될 것이다.

## 이미지 비밀번호 설정

우리는 이 이미지를 바로 실행하면 되는데, 이 이미지는 기존에 설정된 비밀번호가 존재한다. 그래서 당장 바로 실행하면 비밀번호를 몰라 당황하게 될 것이다. 즉, 미리 비밀번호를 설정하는 과정이 필요하다.  
우리가 필요한 커맨드는 **virt-customize**로, 이를 설치하기 위해선 다음의 명령어 입력이 필요하다.

```terminal
sudo apt install libguestfs-tools
```

설치가 끝났다면, 바로 실행시키지 말고 export로 인자를 미리 설정해주어야 한다. 다음 명령어를 입력해주자. 그 다음 우리의 virt-customize 명령어를 입력해준다. 마지막 password: 뒤에 원하는 비밀번호를 넣어주면 된다.  

```terminal
export LIBGUESTFS_DEBUG=1 LIBGUESTFS_TRACE=1
sudo virt-customize -a ubuntu-21.10-server-cloudimg-arm64.img --root-password password:1234
```

이를 통해 준비는 끝났다.  

## qemu 실행

이제 실행만을 앞두고 있다. 명령어는 다음과 같다.

```terminal
qemu-system-aarch64 \
-nographic \
-machine virt,gic-version=max,mte=on \
-m 2G \
-cpu max \
-smp 4 \
-netdev user,id=vnet,hostfwd=:127.0.0.1:0-:22 \
-device virtio-net-pci,netdev=vnet \
-drive file=ubuntu-21.10-server-cloudimg-arm64.img,if=none,id=drive1,cache=writeback \
-device virtio-blk,drive=drive1,bootindex=1 \
-drive file=flash0.img,format=raw,if=pflash \
-drive file=flash1.img,format=raw,if=pflash
```

mte=on으로 우리의 실습을 위해 켜주었고, -m 2G는 메모리 용량이므로 원하는 만큼 설정해주면 된다. -smp도 cpu의 갯수를 설정해주는 옵션이니 마찬가지로 본인 사양에 맞게 설정해주면 된다.  
이제 부팅하면, 아주 긴 화면이 촤라락 지나가고, 잠깐의 기다림이 지나면 로그인 화면이 뜰 것이다. 이제 아이디는 root, 비밀번호는 우리가 위에서 설정한 비밀번호를 입력해주면 잠시 후 root로 접속을 할 수 있을 것이다.  

## 또 설치

지금부터 진행하는 과정은 전부 우리가 에뮬레이팅하는 arm64 환경에서 이루어진다.  
이제 gcc를 설치해주어야 한다. arm64 환경이기에 gcc도 알아서 arm에 맞춰 설치된다. 다음 명령어를 입력하자.  

```terminal
apt-get update
apt-get install -y gcc
```

성공적으로 완료되면, 다음으로 진행하고, 만약 오류가 걸리거나 설치가 제대로 되지 않았다면, 용량 설정의 문제일 수 있으니 참고바란다.  

## MTE 실습

이제 대망의 실습이다. 테스트 코드는 [Memory Tagging Extension (MTE) in AArch64 Linux](https://www.kernel.org/doc/html/latest/arm64/memory-tagging-extension.html) 이 공식 문서의 코드를 그대로 가져왔다. 우리는 이 코드를 그대로 쓰지 않고 prctl, mte의 모드를 결정하는 부분만 변경하였는데, 문서 상에서는 *If multiple modes are specified, the mode is selected as described in the “Per-CPU preferred tag checking modes” section below.*라고 나오며, 공식 문서의 코드는 sync와 async를 동시에 사용하지만 아쉽게도 가상 환경이라 그런지 우리의 실행 환경에서는 오류가 발생한다.  
그러므로 우리는 sync 모드 하나만 사용할 것이며, 자세한 코드의 구성은 아래 코드의 주석을 참고하길 바란다. 코드를 다 살펴보고 어떻게 mte를 체크하는지 확인했다면 아래 명령어를 그대로 입력해주면 된다.

```c
cat > test.c <<EOF
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
EOF
```

이제 test.c라는 새로운 파일이 생성되었을 것이다. 이제 컴파일의 시간이다.  
코드 맨 앞에서 볼 수 있다시피, march=armv8.5-a+memtag와 함께 컴파일해주라는 안내문이 붙어 있다. 그대로 해주자.  

```terminal
gcc -march=armv8.5-a+memtag test.c -o test
```

다 왔다. test를 실행해보자.

```terminal
./test
```

![image](https://user-images.githubusercontent.com/73513005/163797336-fda60fe1-6073-44f1-ae3e-8ecf2c5c8b41.png)

완벽하게 따라했다면, 이처럼 sigsegv가 발생해 segmentation fault를 발생시킴을 확인할 수 있다.  

## 결론

mte를 테스트하는 것보다 가상환경을 세팅하는 것이 더 힘들었다. 거기다가 세팅 후에도 sync와 async가 동시에 켜져있어 발생하는 오류 때문에 한참을 헤맸다. 추후 관련된 작업을 할 때면 훨씬 편하게 할 수 있을 것이다.

reference:  
[What’s new in security for Ubuntu 21.04?](https://ubuntu.com/blog/whats-new-in-security-for-ubuntu-21-04)  
[How to launch ARM aarch64 VM with QEMU from scratch.](https://futurewei-cloud.github.io/ARM-Datacenter/qemu/how-to-launch-aarch64-vm/)  
[cloud image 비밀번호 설정](https://askubuntu.com/questions/451673/default-username-password-for-ubuntu-cloud-image)  
[https://cloud-images.ubuntu.com/releases/21.10/release-20220309/](https://cloud-images.ubuntu.com/releases/21.10/release-20220309/)  

