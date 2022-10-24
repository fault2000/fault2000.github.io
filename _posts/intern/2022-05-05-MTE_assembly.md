---
layout: post
title: MTE assembly
categories: [intern]
tags: [mte, arm, memory safety]
fullview: true
comments: true
author: fault2000
use_math: true
---

[MTE test](https://fault2000.github.io/intern/2022/04/16/MTE_practice.html) 글에서 사용한 테스트에서 사용된 코드의 어셈블리어는 다음과 같다.

```assembly
 0x000000000000096c <+0>:     stp     x29, x30, [sp, #-48]!
   0x0000000000000970 <+4>:     mov     x29, sp
   0x0000000000000974 <+8>:     mov     w0, #0x1e                       // #30
   0x0000000000000978 <+12>:    bl      0x820 <sysconf@plt>
   0x000000000000097c <+16>:    str     x0, [sp, #16]
   0x0000000000000980 <+20>:    mov     x0, #0x1a                       // #26
   0x0000000000000984 <+24>:    bl      0x7c0 <getauxval@plt>
   0x0000000000000988 <+28>:    str     x0, [sp, #24]
   0x000000000000098c <+32>:    ldr     x0, [sp, #24]
   0x0000000000000990 <+36>:    and     x0, x0, #0x40000
   0x0000000000000994 <+40>:    cmp     x0, #0x0
   0x0000000000000998 <+44>:    b.ne    0x9a4 <main+56>  // b.any
   0x000000000000099c <+48>:    mov     w0, #0x1                        // #1
   0x00000000000009a0 <+52>:    b       0xb18 <main+428>
   0x00000000000009a4 <+56>:    mov     w4, #0x0                        // #0
   0x00000000000009a8 <+60>:    mov     w3, #0x0                        // #0
   0x00000000000009ac <+64>:    mov     w2, #0x0                        // #0
   0x00000000000009b0 <+68>:    mov     x1, #0xfff3                     // #65523
   0x00000000000009b4 <+72>:    movk    x1, #0x7, lsl #16
   0x00000000000009b8 <+76>:    mov     w0, #0x37                       // #55
   0x00000000000009bc <+80>:    bl      0x840 <prctl@plt>
   0x00000000000009c0 <+84>:    cmp     w0, #0x0
   0x00000000000009c4 <+88>:    b.eq    0x9dc <main+112>  // b.none
   0x00000000000009c8 <+92>:    adrp    x0, 0x0
   0x00000000000009cc <+96>:    add     x0, x0, #0xbc0
   0x00000000000009d0 <+100>:   bl      0x7a0 <perror@plt>
   0x00000000000009d4 <+104>:   mov     w0, #0x1                        // #1
   0x00000000000009d8 <+108>:   b       0xb18 <main+428>
   0x00000000000009dc <+112>:   mov     x5, #0x0                        // #0
   0x00000000000009e0 <+116>:   mov     w4, #0xffffffff                 // #-1
   0x00000000000009e4 <+120>:   mov     w3, #0x22                       // #34
   0x00000000000009e8 <+124>:   mov     w2, #0x3                        // #3
   0x00000000000009ec <+128>:   ldr     x1, [sp, #16]
   0x00000000000009f0 <+132>:   mov     x0, #0x0                        // #0
   0x00000000000009f4 <+136>:   bl      0x810 <mmap@plt>
   0x00000000000009f8 <+140>:   str     x0, [sp, #32]
   0x00000000000009fc <+144>:   ldr     x0, [sp, #32]
   0x0000000000000a00 <+148>:   cmn     x0, #0x1
   0x0000000000000a04 <+152>:   b.ne    0xa1c <main+176>  // b.any
   0x0000000000000a08 <+156>:   adrp    x0, 0x0
   0x0000000000000a0c <+160>:   add     x0, x0, #0xbd0
   0x0000000000000a10 <+164>:   bl      0x7a0 <perror@plt>
   0x0000000000000a14 <+168>:   mov     w0, #0x1                        // #1
   0x0000000000000a18 <+172>:   b       0xb18 <main+428>
   0x0000000000000a1c <+176>:   mov     w2, #0x23                       // #35
   0x0000000000000a20 <+180>:   ldr     x1, [sp, #16]
   0x0000000000000a24 <+184>:   ldr     x0, [sp, #32]
   0x0000000000000a28 <+188>:   bl      0x850 <mprotect@plt>
   0x0000000000000a2c <+192>:   cmp     w0, #0x0
   0x0000000000000a30 <+196>:   b.eq    0xa48 <main+220>  // b.none
   0x0000000000000a34 <+200>:   adrp    x0, 0x0
   0x0000000000000a38 <+204>:   add     x0, x0, #0xbe0
   0x0000000000000a3c <+208>:   bl      0x7a0 <perror@plt>
   0x0000000000000a40 <+212>:   mov     w0, #0x1                        // #1
   0x0000000000000a44 <+216>:   b       0xb18 <main+428>
   0x0000000000000a48 <+220>:   ldr     x0, [sp, #32]
   0x0000000000000a4c <+224>:   mov     w1, #0x1                        // #1
   0x0000000000000a50 <+228>:   strb    w1, [x0]
   0x0000000000000a54 <+232>:   ldr     x0, [sp, #32]
   0x0000000000000a58 <+236>:   add     x0, x0, #0x1
   0x0000000000000a5c <+240>:   mov     w1, #0x2                        // #2
   0x0000000000000a60 <+244>:   strb    w1, [x0]
   0x0000000000000a64 <+248>:   ldr     x0, [sp, #32]
   0x0000000000000a68 <+252>:   ldrb    w0, [x0]
   0x0000000000000a6c <+256>:   mov     w1, w0
   0x0000000000000a70 <+260>:   ldr     x0, [sp, #32]
   0x0000000000000a74 <+264>:   add     x0, x0, #0x1
   0x0000000000000a78 <+268>:   ldrb    w0, [x0]
   0x0000000000000a7c <+272>:   mov     w2, w0
   0x0000000000000a80 <+276>:   adrp    x0, 0x0
   0x0000000000000a84 <+280>:   add     x0, x0, #0xbf8
   0x0000000000000a88 <+284>:   bl      0x830 <printf@plt>
   0x0000000000000a8c <+288>:   ldr     x0, [sp, #32]
   0x0000000000000a90 <+292>:   irg     x0, x0
   0x0000000000000a94 <+296>:   str     x0, [sp, #40]
   0x0000000000000a98 <+300>:   ldr     x0, [sp, #40]
   0x0000000000000a9c <+304>:   str     x0, [sp, #32]
   0x0000000000000aa0 <+308>:   ldr     x0, [sp, #32]
   0x0000000000000aa4 <+312>:   stg     x0, [x0]
   0x0000000000000aa8 <+316>:   ldr     x1, [sp, #32]
   0x0000000000000aac <+320>:   adrp    x0, 0x0
   0x0000000000000ab0 <+324>:   add     x0, x0, #0xc18
   0x0000000000000ab4 <+328>:   bl      0x830 <printf@plt>
   0x0000000000000ab8 <+332>:   ldr     x0, [sp, #32]
   0x0000000000000abc <+336>:   mov     w1, #0x3                        // #3
   0x0000000000000ac0 <+340>:   strb    w1, [x0]
   0x0000000000000ac4 <+344>:   ldr     x0, [sp, #32]
   0x0000000000000ac8 <+348>:   ldrb    w0, [x0]
   0x0000000000000acc <+352>:   mov     w1, w0
   0x0000000000000ad0 <+356>:   ldr     x0, [sp, #32]
   0x0000000000000ad4 <+360>:   add     x0, x0, #0x1
   0x0000000000000ad8 <+364>:   ldrb    w0, [x0]
   0x0000000000000adc <+368>:   mov     w2, w0
   0x0000000000000ae0 <+372>:   adrp    x0, 0x0
   0x0000000000000ae4 <+376>:   add     x0, x0, #0xbf8
   0x0000000000000ae8 <+380>:   bl      0x830 <printf@plt>
   0x0000000000000aec <+384>:   adrp    x0, 0x0
   0x0000000000000af0 <+388>:   add     x0, x0, #0xc20
   0x0000000000000af4 <+392>:   bl      0x800 <puts@plt>
   0x0000000000000af8 <+396>:   ldr     x0, [sp, #32]
   0x0000000000000afc <+400>:   add     x0, x0, #0x10
   0x0000000000000b00 <+404>:   mov     w1, #0xffffffdd                 // #-35
   0x0000000000000b04 <+408>:   strb    w1, [x0]
   0x0000000000000b08 <+412>:   adrp    x0, 0x0
   0x0000000000000b0c <+416>:   add     x0, x0, #0xc38
   0x0000000000000b10 <+420>:   bl      0x800 <puts@plt>
   0x0000000000000b14 <+424>:   mov     w0, #0x1                        // #1
   0x0000000000000b18 <+428>:   ldp     x29, x30, [sp], #48
   0x0000000000000b1c <+432>:   ret
End of assembler dump.
```

사실 분석할 것도 별로 없이, 코드에 명시되어있는 어셈블리어의  
asm("irg %0, %1" : "=r" (__val) : "r" (ptr));  
asm volatile("stg %0, [%0]" : : "r" (tagged_addr) : "memory");  
를 각각  
0x0000000000000a90 <+292>:   irg     x0, x0  
0x0000000000000aa4 <+312>:   stg     x0, [x0]
에서 찾아볼 수 있다. 여기서 각 어셈블리는 mte만을 위해 추가된 명령어들로, 각각 irg는 포인터에 value(태그값)을 새겨넣는 명령어이고, stg는 메모리에 x0의 값을 새겨넣는 과정이다. 이 과정을 거침으로써 포인터와 그 포인터가 가리키는 메모리의 태그값이 같아지고, 접근해도 오류를 뱉지 않지만, 다른 태그값을 가진 다른 메모리에 접근하면 오류를 발생시킨다.  
이 테스트 코드를 보면 알 수 있듯, 기본적으로 모든 초기 포인터나 메모리의 tag 값은 0으로 되어 있어 따로 태그값을 설정하지 않는 이상 mte는 작동하지 않는다. 하지만 포인터나 메모리에 태그값을 새로 할당하는 순간 그 포인터나 메모리는 같은 태그값으로만 접근할 수 있는 것이다.