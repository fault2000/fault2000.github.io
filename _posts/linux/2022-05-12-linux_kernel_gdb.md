---
layout: post
title: ARM Linux kernel debugging
category: [linux]
tags: [linux, kernel, arm, gdb]
fullview: true
comments: true
use_math: true
author: fault2000
---

이 글에서는 qemu, gdb를 통해 ARM 버전의 linux kernel을 디버깅하기 위한 과정들을 살펴보고자한다.  

## Tools

가장 먼저 커널을 빌드하는데 필요한 약간의 툴들을 받는다.

```
sudo apt-get install -y gcc-aarch64-linux-gnu libncurses5-dev build-essential flex bison bc git libssl-dev qemu qemu-system gdb-multiarch
```

## Linux Kernel

그 다음 linux kernel을 다운받아야한다. kernel을 다운받는다. 아래 명령어를 입력하면 그 위치에 리눅스가 받아질 것이다.  

```
git clone https://github.com/torvalds/linux.git
```

## Kernel Build

이제 kernel build를 위해 설정을 몇 개 만져주어야한다.  

```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

필자의 목적은 MTE의 작동을 확인하기 위한 것이므로, arm64 버전과 크로스 컴파일을 따로 인자로 추가해주었다. 크로스 컴파일을 추가해주지 않으면 MTE 옵션이 커널 설정에 없을 수 있으므로 주의바란다.  
아무튼 커널 디버깅을 위해 만져주어야하는 옵션이 있는데, 초기화면부터 Kernel Hacking/Compile time checks and compiler option/Debug information (Disable debug information) --> 메뉴가 있는데, 이를 선택하여 Rely on the toolchain's implicit default DWARF version으로 바꾸어주어야 하며, 아래 Reduce debugging information가 체크되어 있다면 체크해제를 해주길 권한다.  
---
이제 남은 건 build뿐이다. 다음 명령어를 입력해 빌드를 시작한다. 이 과정은 상당한 시간이 소요되므로 본인 컴퓨터에서 사용할 수 있는 코어 수를 확인하여 -j4(4코어 사용)등으로 코어 수를 지정해 더 빨리 빌드할 수 있으니 참고바란다.

```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4 all
```

## QEMU

이제 빌드 과정으로 얻은 Image를 통해 커널을 실행시켜볼 차례이다. 아래 명령어를 통해 실행할 수 있다.  

```
qemu-system-aarch64 -kernel arch/arm64/boot/Image -cpu cortex-a57 -m 1024M -M virt -nographic -smp 2 -append "console=ttyAMA0"
```

위 옵션들 몇 개를 설명하자면, -kernel은 우리가 build한 Image이고, -m은 qemu에 할당할 램의 크기를, -smp는 할당할 코어수를 나타낸다. 알아둬야 할 부분은 이게 끝이다.  
그러나 당장 실행하면 커널 패닉이 일어난다. 이유는 우리가 루트 디렉토리를 따로 만들어주지 않아서이다. 일단 종료하고 이 커널 패닉을 고칠 rootfs 디렉토리를 만들러가보자.  

## Busybox

[busybox 홈페이지](https://busybox.net/)로 이동해 busybox를 다운받아준다. 필자는 가장 최신의 안정적인 버전인 1.33.1을 받았으며, 아마 그 이하 버전을 받지 않는 이상 문제는 없을 것이다.  
리눅스 파일과 다른 곳에 압축을 풀어준 뒤 압축을 푼 파일 디렉토리로 이동한다. 그 디렉토리에서 다음과 같은 명령어를 입력한다.

```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4 menuconfig
```

친숙한 명령어인만큼 전과 비슷한 화면이 뜬다. 여기서 우리가 건드려주어야 하는 옵션은 하나로, Setting의

```
--- Build Options
[*] Build static binary (no shared libs)
```

를 체크해주어야한다.  
그 후 빌드를 진행해준다.

```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4
```

빌드가 끝났다면, 설치를 진행한다.

```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4 install
```

이 과정이 성공적으로 끝났다면, 이제 _install 디렉토리로 이동한다. 그러면 속에 bin, linuxrc, sbin, usr 디렉토리와 파일들이 보일 것이다. 이제 우리는 몇 가지 파일들을 추가해주어야한다.

```
cd _install
mkdir -p dev etc home lib mnt proc root sys tmp var
```

이제 세부적인 설정을 몇 가지를 직접 해주어야한다. 아래 과정들은 전부 busybox 폴더 속 _install 디렉토리에서 수행된다.  

### etc/inittab

```
vi etc/inittab
```

inittab 파일을 만들고 그 안에 다음을 넣는다.  

```
::sysinit:/etc/init.d/rcS
::respawn:-/bin/sh
::askfirst:-/bin/sh
::cttlaltdel:/bin/umount -a -r
```

마무리로 권한 설정까지 해주면 이 파일은 끝

```
chmod 755 etc/inittab
```

### etc/init.d/rcS

위 과정과 동일하다. 마찬가지로 init.d 폴더를 etc 안에 만들고, 그 안에 만든 rcS 속에 내용을 입력해주면 끝. 

```
mkdir -p etc/init.d/
vi etc/init.d/rcS
```

내용은 다음과 같다.

```
echo "----------mount all in fstab----------------"
/bin/mount -a #  /etc/fstab，      
mkdir -p /dev/pts
mount -t devpts devpts /dev/pts
echo /sbin/mdev > /proc/sys/kernel/hotplug
mdev -s
```

권한 설정도 잊지 말자.

```
chmod 755 etc/init.d/rcS
```

### etc/fstab

설명 생략

```
vi etc/fstab
```

내용은 다음과 같다.

```
#device mount-point type option dump fsck
proc  /proc proc  defaults 0 0
temps /tmp  rpoc  defaults 0 0
none  /tmp  ramfs defaults 0 0
sysfs /sys  sysfs defaults 0 0
mdev  /dev  ramfs defaults 0 0
```

### dev

dev 폴더에서 따로 진행된다.

```
cd dev
sudo mknod console c 5 1
sudo mknod null c 1 3
cd ..
```

### rootfs.cpio.gz

이제 결과물을 모아 압축해준다. 이 과정도 _install 디렉토리에서 실행된다.

```
find . | cpio -o -H newc > rootfs.cpio 
gzip -c rootfs.cpio > rootfs.cpio.gz
```

이 과정이 끝나면, _install 디렉토리 안에 rootfs.cpio.gz 파일이 생성되었을 것이다. 이를 리눅스 커널로 옮긴다. 그 다음 리눅스 커널 디렉토리로 돌아가자.  

## QEMU

앞에서 수행한 busybox 과정은 그냥 커널을 실행하면 root 파일이 없는 관계로 일어나는 kernel panic을 피하기 위해 진행되었다. 이제 문제를 해결했으니, 한번 실행시켜보자.  

```
qemu-system-aarch64 \
-machine virt \
-cpu cortex-a57 \
-machine type=virt \
-nographic \
-smp 2 \
-m 2048 \
-kernel ./arch/arm64/boot/Image \
-append "rdinit=/linuxrc console=ttyAMA0 nokaslr" \
-initrd ./rootfs.cpio.gz
```

알아두어야할 점은 -smp가 코어수, -m 이 메모리 할당량을 나타내니 각자 상황에 맞춰 조절해주면 되겠다.  

실행을 하면 여러 줄의 관련 코드가 뜬 후, 마지막 부분은 아래와 같이 나올 것이다.

![image](https://user-images.githubusercontent.com/73513005/168614822-6632005f-f2c8-4272-b9c5-e9bb62b73f64.png)

아마 필자가 참고한 블로그와 버전이 달라지면서 init.d/rcS에 담긴 내용에서 오류가 나는 것처럼 보이는데, 중요하진 않으므로 넘어가자.  
엔터를 눌러주면 콘솔이 활성화되고, 시험삼아 uname -a를 입력하면 다음과 같이 뜨면서 커널이 잘 빌드되었음을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/73513005/168615298-00173fa9-e95c-486a-924f-4272728d3704.png)

## GDB

이제 커널 빌드가 끝났으니 gdb를 연결하여 debugging할 준비를 해보자. 

```
qemu-system-aarch64 \
-machine virt \
-cpu cortex-a57 \
-machine type=virt \
-nographic \
-smp 2 \
-m 2048 \
-kernel ./arch/arm64/boot/Image \
-append "rdinit=/linuxrc console=ttyAMA0 nokaslr" \
-initrd ./rootfs.cpio.gz \
-s -S
```

위 명령어는 새로 -s, -S가 추가되었는데, 각각 -s는 gdb를 기본으로 쓰되, tcp 포트 1234로 연결하겠다는 옵션이고, -S는 cpu가 시작해줄때까지 정지되어있는 옵션이다.  
그래서 실제로 실행해보면, 당장은 아무런 반응없이 멈춰있음을 볼 수 있다.  
이제 새로운 커맨드창을 열고, 리눅스 커널 디렉토리로 다시 이동한다. 여기서 다음 명령어를 입력해준다.  

```
gdb-multiarch ./vmlinux
```

그러면 평소 우리가 보던 gdb 창이 뜰텐데, 다음 명령어를 입력한다.  

```
target remote localhost:1234
```

그러면 다음과 같이 나올 것이다.  

![image](https://user-images.githubusercontent.com/73513005/168619512-53978033-9b9f-4490-b9a7-feb1d81d4cb9.png)  

이제 break point를 잡자, 다들 start_kernel에 잡는 것을 기본으로 하니 우리도 동일하게 진행한다.  

```
b start_kernel
hb start_kernel
```

다른 분들이 하는 것처럼 그냥 break만 걸면 멈추지 않아 하드웨어 breakpoint를 따로 걸어주었다.  

![image](https://user-images.githubusercontent.com/73513005/168622469-b9002296-a830-4dd3-bfcb-5f17255becab.png)

그러면 우리가 원하는대로 start_kernel에 멈춰있는 커널을 확인할 수 있다.  
이번 포스트는 여기까지 진행하고, 다음엔 이제 커널에서 MTE를 활용하는 구간이 있는지 확인하는 포스트를 진행하도록 하겠다.