---
layout: post
title: root-me Cracking ELF x86 - 0 protection
category: [root-me]
tags: [root-me, Cracking, CTF]
fullview: true
comments: true
use_math: true
author: fault2000
---

이번 문제를 시작으로 root-me의 몇 문제들을 풀어보려고 한다.  

먼저 도전 시작하기를 누르면 ch1.bin 파일을 다운로드한다. .bin 파일은 바이너리 파일이므로 간편하게 보기 위해 ida pro를 통해 열어보았다.  

<img width="866" alt="image" src="https://user-images.githubusercontent.com/73513005/193829566-08267672-d32d-4366-b5ae-04fc4ae19312.png">  

아무래도 위 사진처럼 계속 보기엔 가독성이 떨어지므로, 아래를 참고해주면 좋겠다.  

```
; int __cdecl main(int argc, const char **argv, const char **envp)
.text:0804869D                 public main
.text:0804869D main            proc near               ; DATA XREF: _start+17↑o
.text:0804869D
.text:0804869D var_C           = dword ptr -0Ch
.text:0804869D var_8           = dword ptr -8
.text:0804869D argc            = dword ptr  8
.text:0804869D argv            = dword ptr  0Ch
.text:0804869D envp            = dword ptr  10h
.text:0804869D
.text:0804869D                 lea     ecx, [esp+4]
.text:080486A1                 and     esp, 0FFFFFFF0h
.text:080486A4                 push    dword ptr [ecx-4]
.text:080486A7                 push    ebp
.text:080486A8                 mov     ebp, esp
.text:080486AA                 push    ecx
.text:080486AB                 sub     esp, 24h
.text:080486AE                 mov     [ebp+var_8], offset a123456789 ; "123456789"
.text:080486B5                 mov     dword ptr [esp], offset asc_804884C ; "#######################################"...
.text:080486BC                 call    _puts
.text:080486C1                 mov     dword ptr [esp], offset aBienvennueDans ; "##        Bienvennue dans ce challenge "...
.text:080486C8                 call    _puts
.text:080486CD                 mov     dword ptr [esp], offset asc_80488CC ; "#######################################"...
.text:080486D4                 call    _puts
.text:080486D9                 mov     dword ptr [esp], offset aVeuillezEntrer ; "Veuillez entrer le mot de passe : "
.text:080486E0                 call    _printf
.text:080486E5                 mov     eax, [ebp+var_C]
.text:080486E8                 mov     [esp], eax
.text:080486EB                 call    getString
.text:080486F0                 mov     [ebp+var_C], eax
.text:080486F3                 mov     eax, [ebp+var_8]
.text:080486F6                 mov     [esp+4], eax
.text:080486FA                 mov     eax, [ebp+var_C]
.text:080486FD                 mov     [esp], eax
.text:08048700                 call    _strcmp
.text:08048705                 test    eax, eax
.text:08048707                 jnz     short loc_804871E
.text:08048709                 mov     eax, [ebp+var_8]
.text:0804870C                 mov     [esp+4], eax
.text:08048710                 mov     dword ptr [esp], offset aBienJoueVousPo ; "Bien joue, vous pouvez valider l'epreuv"...
.text:08048717                 call    _printf
.text:0804871C                 jmp     short loc_804872A
.text:0804871E ; ---------------------------------------------------------------------------
.text:0804871E
.text:0804871E loc_804871E:                            ; CODE XREF: main+6A↑j
.text:0804871E                 mov     dword ptr [esp], offset aDommageEssayeE ; "Dommage, essaye encore une fois."
.text:08048725                 call    _puts
.text:0804872A
.text:0804872A loc_804872A:                            ; CODE XREF: main+7F↑j
.text:0804872A                 mov     eax, 0
.text:0804872F                 add     esp, 24h
.text:08048732                 pop     ecx
.text:08048733                 pop     ebp
.text:08048734                 lea     esp, [ecx-4]
.text:08048737                 retn
.text:08048737 main            endp
```

흠. 길다. 그러니 중요한 부분만 잘라 설명하겠다.  

```
.text:080486AE                 mov     [ebp+var_8], offset a123456789 ; "123456789"
.text:080486B5                 mov     dword ptr [esp], offset asc_804884C ; "#######################################"...
.text:080486BC                 call    _puts
.text:080486C1                 mov     dword ptr [esp], offset aBienvennueDans ; "##        Bienvennue dans ce challenge "...
.text:080486C8                 call    _puts
.text:080486CD                 mov     dword ptr [esp], offset asc_80488CC ; "#######################################"...
.text:080486D4                 call    _puts
.text:080486D9                 mov     dword ptr [esp], offset aVeuillezEntrer ; "Veuillez entrer le mot de passe : "
.text:080486E0                 call    _printf
```

먼저 123456789를 ebp+var_8, 즉 ebp-8의 위치에 저장함을 기억하자. 처음에 보고 에이 이게 비밀번호일리가 없잖아 라며 넘어갔다.  
그 다음 아마 줄 구분용 줄 인 것 같은 #과 안에는 글이 있는데, 킹갓 파파고님께서 프랑스어란다. 대충 내용은 이 도전에 다시 온 걸 환영한다며...? 초면인데, 아무튼 아래 줄은 암호를 입력하라는 글이다.  

```
.text:080486E5                 mov     eax, [ebp+var_C]
.text:080486E8                 mov     [esp], eax
.text:080486EB                 call    getString
.text:080486F0                 mov     [ebp+var_C], eax
.text:080486F3                 mov     eax, [ebp+var_8]
.text:080486F6                 mov     [esp+4], eax
.text:080486FA                 mov     eax, [ebp+var_C]
.text:080486FD                 mov     [esp], eax
.text:08048700                 call    _strcmp
```

이제 아래에는 ebp+var_C에 getString의 결과를 저장하고, ebp+var_8과 ebp+var_C를 각각 가져와 비교하는... 잠깐, ebp+var_8의 내용은 123456789 였는데? 그럼 이게 비밀번호인가?  

그래서 한 번 사이트에 입력해주었다. 그랬더니 풀리더라. 어흑마이깟