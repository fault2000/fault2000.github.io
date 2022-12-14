---
layout: post
title: OverTheWire Leviathan walkthrough
category: [OverTheWire, linux]
tags: [OverTheWire, Leviathan]
fullview: true
comments: true
use_math: true
author: fault2000
---

# Leviathan

시작 전에, 비밀번호가 자신의 것과 같지 않을 수 있는데, 아마 overthewire 측이 비밀번호를 계속해서 바꾸는 모양이다. 그러니 꼼수 쓸 생각하지말고 직접 해보자.

## Leviathan 0 → 1

 접속 후 ls 시 숨겨진 파일과 디렉토리만 존재한다. 이 중 수상해보이는 디렉토리인 .backup에 들어가면 bookmarks.html이라는 파일만 존재한다.

![https://user-images.githubusercontent.com/73513005/206913909-2bf48dd4-7080-4697-830e-e4ab0a59173e.png](https://user-images.githubusercontent.com/73513005/206913909-2bf48dd4-7080-4697-830e-e4ab0a59173e.png)

 이를 직접 열어보면 파일 내용이 너무 많다. 비밀번호가 있는지 확인하기 위해 grep으로 키워드를 검색해보면 답을 구할 수 있다.

![https://user-images.githubusercontent.com/73513005/206914102-60a625e7-79b9-4f80-9b67-f98cd1ed8f9d.png](https://user-images.githubusercontent.com/73513005/206914102-60a625e7-79b9-4f80-9b67-f98cd1ed8f9d.png)

password: PPIfmI1qsA

## Leviathan 1 → 2

![https://user-images.githubusercontent.com/73513005/206915230-ffcdf8c5-eed9-4377-a37e-a658f8219d42.png](https://user-images.githubusercontent.com/73513005/206915230-ffcdf8c5-eed9-4377-a37e-a658f8219d42.png)

 check라는 실행 파일이 존재하고, 이를 실행해본 결과 패스워드를 입력받은 다음, 무언가랑 비교하는 듯 하다. gdb-peda를 통해 확인해보았다.

```nasm
0x080491e6 <+0>:	lea    0x4(%esp),%ecx
   0x080491ea <+4>:	and    $0xfffffff0,%esp
   0x080491ed <+7>:	push   -0x4(%ecx)
   0x080491f0 <+10>:	push   %ebp
   0x080491f1 <+11>:	mov    %esp,%ebp
   0x080491f3 <+13>:	push   %ebx
   0x080491f4 <+14>:	push   %ecx
=> 0x080491f5 <+15>:	sub    $0x20,%esp
   0x080491f8 <+18>:	mov    %gs:0x14,%eax
   0x080491fe <+24>:	mov    %eax,-0xc(%ebp)
   0x08049201 <+27>:	xor    %eax,%eax
   0x08049203 <+29>:	movl   $0x786573,-0x20(%ebp)
   0x0804920a <+36>:	movl   $0x72636573,-0x13(%ebp)
   0x08049211 <+43>:	movw   $0x7465,-0xf(%ebp)
   0x08049217 <+49>:	movb   $0x0,-0xd(%ebp)
   0x0804921b <+53>:	movl   $0x646f67,-0x1c(%ebp)
   0x08049222 <+60>:	movl   $0x65766f6c,-0x18(%ebp)
   0x08049229 <+67>:	movb   $0x0,-0x14(%ebp)
   0x0804922d <+71>:	sub    $0xc,%esp
   0x08049230 <+74>:	push   $0x804a008
   0x08049235 <+79>:	call   0x8049060 <printf@plt>
   0x0804923a <+84>:	add    $0x10,%esp
   0x0804923d <+87>:	call   0x8049070 <getchar@plt>
   0x08049242 <+92>:	mov    %al,-0x24(%ebp)
   0x08049245 <+95>:	call   0x8049070 <getchar@plt>
   0x0804924a <+100>:	mov    %al,-0x23(%ebp)
   0x0804924d <+103>:	call   0x8049070 <getchar@plt>
   0x08049252 <+108>:	mov    %al,-0x22(%ebp)
   0x08049255 <+111>:	movb   $0x0,-0x21(%ebp)
   0x08049259 <+115>:	sub    $0x8,%esp
   0x0804925c <+118>:	lea    -0x20(%ebp),%eax
   0x0804925f <+121>:	push   %eax
   0x08049260 <+122>:	lea    -0x24(%ebp),%eax
   0x08049263 <+125>:	push   %eax
   0x08049264 <+126>:	call   0x8049040 <strcmp@plt>
   0x08049269 <+131>:	add    $0x10,%esp
   0x0804926c <+134>:	test   %eax,%eax
   0x0804926e <+136>:	jne    0x804929b <main+181>
   0x08049270 <+138>:	call   0x8049090 <geteuid@plt>
   0x08049275 <+143>:	mov    %eax,%ebx
   0x08049277 <+145>:	call   0x8049090 <geteuid@plt>
   0x0804927c <+150>:	sub    $0x8,%esp
   0x0804927f <+153>:	push   %ebx
   0x08049280 <+154>:	push   %eax
   0x08049281 <+155>:	call   0x80490c0 <setreuid@plt>
   0x08049286 <+160>:	add    $0x10,%esp
   0x08049289 <+163>:	sub    $0xc,%esp
   0x0804928c <+166>:	push   $0x804a013
   0x08049291 <+171>:	call   0x80490b0 <system@plt>
   0x08049296 <+176>:	add    $0x10,%esp
   0x08049299 <+179>:	jmp    0x80492ab <main+197>
   0x0804929b <+181>:	sub    $0xc,%esp
   0x0804929e <+184>:	push   $0x804a01b
   0x080492a3 <+189>:	call   0x80490a0 <puts@plt>
   0x080492a8 <+194>:	add    $0x10,%esp
   0x080492ab <+197>:	mov    $0x0,%eax
   0x080492b0 <+202>:	mov    -0xc(%ebp),%edx
--Type <RET> for more, q to quit, c to continue without paging--
   0x080492b3 <+205>:	sub    %gs:0x14,%edx
   0x080492ba <+212>:	je     0x80492c1 <main+219>
   0x080492bc <+214>:	call   0x8049080 <__stack_chk_fail@plt>
   0x080492c1 <+219>:	lea    -0x8(%ebp),%esp
   0x080492c4 <+222>:	pop    %ecx
   0x080492c5 <+223>:	pop    %ebx
   0x080492c6 <+224>:	pop    %ebp
   0x080492c7 <+225>:	lea    -0x4(%ecx),%esp
   0x080492ca <+228>:	ret
```

어셈블리는 다음과 같은데, cmp 부분에 break를 걸고 비교되는 값을 확인해보았다.

![https://user-images.githubusercontent.com/73513005/206915395-dd8dfa1b-ea11-4080-a457-b32c31873576.png](https://user-images.githubusercontent.com/73513005/206915395-dd8dfa1b-ea11-4080-a457-b32c31873576.png)

 놀랍게도 비교되는 값이 sex이다… 그러면 값을 입력해보자.

![https://user-images.githubusercontent.com/73513005/206915464-2c0027ff-c5b8-472e-82b8-875fc782723b.png](https://user-images.githubusercontent.com/73513005/206915464-2c0027ff-c5b8-472e-82b8-875fc782723b.png)

 또다른 콘솔창으로 연결되고 유저를 확인해보니 leviathan2로 설정되어있다. 위 어셈블리를 확인해보면 알겠지만 setuid를 통해 유저 권한을 변경한 모양이다.

![https://user-images.githubusercontent.com/73513005/206915618-844d8762-bff8-4678-94c3-d551c7fcb919.png](https://user-images.githubusercontent.com/73513005/206915618-844d8762-bff8-4678-94c3-d551c7fcb919.png)

 사실 비밀번호를 찾기 위해 /etc/passwd랑 /etc/shadow를 뒤져봤는데, 없어서 뭔가 했더니 홈페이지에서 다음과 같이 안내하고 있었다.

> Data for the levels can be found in **the homedirectories**
. You can look at **/etc/leviathan_pass**
 for the various level passwords.
> 

그래서 확인한 결과가 위와 같다.

password: mEh5PNl10e

# Leviatian 2 → 3

![https://user-images.githubusercontent.com/73513005/206924297-440295b2-f19c-4f12-a1e4-dc845873e6a0.png](https://user-images.githubusercontent.com/73513005/206924297-440295b2-f19c-4f12-a1e4-dc845873e6a0.png)

처음 접속하면 저번처럼 printfile이라는 실행 파일이 있다. 

![https://user-images.githubusercontent.com/73513005/206924537-88cc022b-ae76-4ff6-8f1b-eb2dd07a62ac.png](https://user-images.githubusercontent.com/73513005/206924537-88cc022b-ae76-4ff6-8f1b-eb2dd07a62ac.png)

 실행 시 사용 방법을 알려주고, 테스트로 아무 파일이나 줘봤더니 파일 속 내용을 그대로 출력한다. 이름과 같은 실행인가본데, 어떻게 동작하는지 알기 위해 ltrace를 실행해보았다.

![https://user-images.githubusercontent.com/73513005/206924639-40e4758f-f09c-41f1-afe8-fce1fff724ca.png](https://user-images.githubusercontent.com/73513005/206924639-40e4758f-f09c-41f1-afe8-fce1fff724ca.png)

출력 내용 때문에 쓸데없이 커보이는데, 중요한 점은 위의 내용들이다. 먼저 access 함수를 통해 유저가 권한이 있는지 확인한다. 여기서 유저는 leviathan2이다. 그 다음 /bin/cat 과 printfile의 인자로 줬던 값을 합쳐 문자열을 만들어둔다. 즉 cat 명령을 만들어두는 것이다. 이제 geteuid를 통해 가져온 leviathan3의 권한을 자신의 권한으로 설정한 다음, leviathan3의 권한으로 /bin/cat을 실행한다.

 함수 설명은 충분하고, 이제 어떻게 해야할까. 먼저 /bin/cat 명령 문자열이 만들어지는 과정을 보면 우리가 준 값을 그냥 가져다 붙여쓴다. 즉 띄어쓰기가 있는 파일을 가져다 주면 그 띄어쓰기된 문자열이 분리된 두 파일이라고 착각하는 것이다.

![https://user-images.githubusercontent.com/73513005/206925153-f4b1a011-379e-4ffe-bbc8-d5787718063b.png](https://user-images.githubusercontent.com/73513005/206925153-f4b1a011-379e-4ffe-bbc8-d5787718063b.png)

 위는 설명을 위해 준비한 예제이다. 먼저 “asd qwe”라는 파일을 만들어주고 그와 동시에 asd라는 파일도 만들어주었다. 그 다음 printfile 명령을 실행하니 먼저 “asd qwe”의 권한을 확인한 함수는 그 다음 문자열을 “/bin/cat asd qwe”라고 만들었고, 이에 명령이 실행되면서 “asd”, “qwe” 각각 출력을 실행한 것이다.

 그럼 이제 대망의 풀이이다. 심볼릭 링크를 통해 qwe를 /etc/leviathan_pass/leviathan3에 연결하고 위 명령을 그대로 실행해주자.

![https://user-images.githubusercontent.com/73513005/206925352-2a614ab8-387b-4b75-8729-9404c3440cc2.png](https://user-images.githubusercontent.com/73513005/206925352-2a614ab8-387b-4b75-8729-9404c3440cc2.png)

 비밀번호를 얻을 수 있다.

 password: Q0G8j4sakn

## Leviathan 3 → 4

![https://user-images.githubusercontent.com/73513005/207032262-b190b630-69b5-406d-a580-8bd8ea7c9880.png](https://user-images.githubusercontent.com/73513005/207032262-b190b630-69b5-406d-a580-8bd8ea7c9880.png)

접속 시 level3라는 실행 파일이 있고, 직접 실행해보면 패스워드를 받은 후 무언가랑 비교하는 것 같다. ltrace를 통해 한번 내부 실행 과정을 보자.

![https://user-images.githubusercontent.com/73513005/207032587-37221df5-f5e5-41ad-b57c-0851bd432c48.png](https://user-images.githubusercontent.com/73513005/207032587-37221df5-f5e5-41ad-b57c-0851bd432c48.png)

 처음엔 h0no33과 kakaka를 비교하더니, 그 뒤 비밀번호를 받고 이를 snlprintf와 비교한다음 틀렸다는 답을 내놓는다. 그렇다면 우리의 입력값을 snlprintf로 바꿔서 넣어보자.

![https://user-images.githubusercontent.com/73513005/207032882-97dbe6c1-1827-4d41-916a-bbd2efe03dc5.png](https://user-images.githubusercontent.com/73513005/207032882-97dbe6c1-1827-4d41-916a-bbd2efe03dc5.png)

![https://user-images.githubusercontent.com/73513005/207033089-dfab7ff4-fa90-4c6e-8cb0-51f34286c357.png](https://user-images.githubusercontent.com/73513005/207033089-dfab7ff4-fa90-4c6e-8cb0-51f34286c357.png)

 이를 입력하면 쉘을 획득할 수 있고, 권한을 확인하면 leviathan4임을 확인할 수 있다.

 passwd: AgvropI4OA

## Leviathan 4 → 5

![https://user-images.githubusercontent.com/73513005/207034423-3068c3a4-f580-4eb6-b39c-d4ddb530588f.png](https://user-images.githubusercontent.com/73513005/207034423-3068c3a4-f580-4eb6-b39c-d4ddb530588f.png)

딱봐도 수상해보인다. .trash에 들어가보자.

![https://user-images.githubusercontent.com/73513005/207034552-0178b555-f4a1-44de-a890-8e681b365c26.png](https://user-images.githubusercontent.com/73513005/207034552-0178b555-f4a1-44de-a890-8e681b365c26.png)

실행 파일 bin이 딸랑있다. 실행 결과는 다음과 같다.

![https://user-images.githubusercontent.com/73513005/207034675-ea6daaa1-c25c-4e93-8c39-80be118dee0a.png](https://user-images.githubusercontent.com/73513005/207034675-ea6daaa1-c25c-4e93-8c39-80be118dee0a.png)

0과 1로 이루어진 값들을 보니 이진수가 의심된다. 그리고 파일 실행 과정을 보니 이 값은 leviathan5의 비밀번호 파일을 열었음을 확인할 수 있다. 위 이미지를 다시 보면 bin 파일의 소유권이 leviathan5임을 볼 수 있다. 한번 이 값을 문자열로 변환해보았다.

![https://user-images.githubusercontent.com/73513005/207038557-6efa8ec3-21d2-44f2-aff9-78c56e3740ec.png](https://user-images.githubusercontent.com/73513005/207038557-6efa8ec3-21d2-44f2-aff9-78c56e3740ec.png)

그 후 입력해보았더니 정상적으로 접속됨을 확인할 수 있었다.

passwd: EKKlTF1Xqs

## Leviathan 5 → 6

![https://user-images.githubusercontent.com/73513005/207058381-fa263ef3-d4a4-4d2a-b235-bac3aabf4f24.png](https://user-images.githubusercontent.com/73513005/207058381-fa263ef3-d4a4-4d2a-b235-bac3aabf4f24.png)

leviathan6의 권한으로 실행되는 leviathan5 실행 파일이 있다. 실행해보자.

![https://user-images.githubusercontent.com/73513005/207058825-8b37f705-a884-44b7-9783-5d32f2a7f870.png](https://user-images.githubusercontent.com/73513005/207058825-8b37f705-a884-44b7-9783-5d32f2a7f870.png)

/tmp/file.log 파일을 찾을 수 없다고 한다. 대충 내용을 입력하고 실행해보니 다음과 같았다.

![https://user-images.githubusercontent.com/73513005/207059386-0bb68663-92ad-4118-8a0d-0aad14ea9fa3.png](https://user-images.githubusercontent.com/73513005/207059386-0bb68663-92ad-4118-8a0d-0aad14ea9fa3.png)

vi 과정에서 내용으로 hello를 입력했었다. 보아하니 내용을 그대로 가져와서 출력하는 과정을 담았음을 유추할 수 있는데, ltrace를 그냥 바로 실행하니 파일이 없다고 나오고, 원래 있던 내용도 사라져있음을 볼 수 있었다. 그래서 다시 /tmp/file.log를 만들어주고 ltrace로 분석해보았다.

![https://user-images.githubusercontent.com/73513005/207060062-8027a598-448b-4074-b505-d4c39fba02eb.png](https://user-images.githubusercontent.com/73513005/207060062-8027a598-448b-4074-b505-d4c39fba02eb.png)

간략히 보자면 먼저 /tmp/file.log를 가져온 후 파일 내용이 없을 때까지 char를 하나씩 가져와 출력한다. 그 뒤 파일의 끝에 도달하면 현재 유저(leviathan5)의 권한을 가져와 /tmp/file.log를 삭제한다. 어쩐지 두 번 실행이 안 되더니 다른 유저가 와서 그냥 실행했는데 문제가 풀리는 상황을 방지하기 위해 파일 삭제 과정을 넣었나보다.

 해결책은 단순하다. 심볼릭 링크를 통해 leviathan6를 /tmp/file.log에 연결한다. 그러면 leviathan5 실행 파일의 사용자 권한은 leviathan6이므로 이를 읽을 수 있을 것이다. 확인해보자.

![https://user-images.githubusercontent.com/73513005/207063268-dc1f1972-41f3-4026-ad1c-7ba2625c4f70.png](https://user-images.githubusercontent.com/73513005/207063268-dc1f1972-41f3-4026-ad1c-7ba2625c4f70.png)

이를 통해 비밀번호를 알아낼 수 있다.

passwd: YZ55XPVk2l

## Leviathan 6 → 7

![https://user-images.githubusercontent.com/73513005/207066196-4987d620-55a2-41cc-b11f-ba5f660f2591.png](https://user-images.githubusercontent.com/73513005/207066196-4987d620-55a2-41cc-b11f-ba5f660f2591.png)

일단 또 leviathan7의 uid를 가지는 leviathan6 실행 파일이 있다. 실행 결과는 다음과 같다.

![https://user-images.githubusercontent.com/73513005/207066988-53416dc1-f9ce-4c6a-a710-1bb9f1395346.png](https://user-images.githubusercontent.com/73513005/207066988-53416dc1-f9ce-4c6a-a710-1bb9f1395346.png)

4개의 정수 코드를 받음을 볼 수 있다. ltrace로 봐도 별 게 안 보이니 그냥 peda로 살펴보았다.

```nasm
Dump of assembler code for function main:
   0x080491d6 <+0>:     lea    ecx,[esp+0x4]
   0x080491da <+4>:     and    esp,0xfffffff0
   0x080491dd <+7>:     push   DWORD PTR [ecx-0x4]
   0x080491e0 <+10>:    push   ebp
   0x080491e1 <+11>:    mov    ebp,esp
   0x080491e3 <+13>:    push   ebx
   0x080491e4 <+14>:    push   ecx
   0x080491e5 <+15>:    sub    esp,0x10
   0x080491e8 <+18>:    mov    eax,ecx
   0x080491ea <+20>:    mov    DWORD PTR [ebp-0xc],0x1bd3
   0x080491f1 <+27>:    cmp    DWORD PTR [eax],0x2
   0x080491f4 <+30>:    je     0x8049216 <main+64>
   0x080491f6 <+32>:    mov    eax,DWORD PTR [eax+0x4]
   0x080491f9 <+35>:    mov    eax,DWORD PTR [eax]
   0x080491fb <+37>:    sub    esp,0x8
   0x080491fe <+40>:    push   eax
   0x080491ff <+41>:    push   0x804a008
   0x08049204 <+46>:    call   0x8049050 <printf@plt>
   0x08049209 <+51>:    add    esp,0x10
   0x0804920c <+54>:    sub    esp,0xc
   0x0804920f <+57>:    push   0xffffffff
   0x08049211 <+59>:    call   0x8049090 <exit@plt>
   0x08049216 <+64>:    mov    eax,DWORD PTR [eax+0x4]
   0x08049219 <+67>:    add    eax,0x4
   0x0804921c <+70>:    mov    eax,DWORD PTR [eax]
   0x0804921e <+72>:    sub    esp,0xc
   0x08049221 <+75>:    push   eax
=> 0x08049222 <+76>:    call   0x80490b0 <atoi@plt>
   0x08049227 <+81>:    add    esp,0x10
   0x0804922a <+84>:    cmp    DWORD PTR [ebp-0xc],eax
   0x0804922d <+87>:    jne    0x804925a <main+132>
   0x0804922f <+89>:    call   0x8049060 <geteuid@plt>
   0x08049234 <+94>:    mov    ebx,eax
   0x08049236 <+96>:    call   0x8049060 <geteuid@plt>
   0x0804923b <+101>:   sub    esp,0x8
   0x0804923e <+104>:   push   ebx
   0x0804923f <+105>:   push   eax
   0x08049240 <+106>:   call   0x80490a0 <setreuid@plt>
   0x08049245 <+111>:   add    esp,0x10
   0x08049248 <+114>:   sub    esp,0xc
   0x0804924b <+117>:   push   0x804a022
   0x08049250 <+122>:   call   0x8049080 <system@plt>
   0x08049255 <+127>:   add    esp,0x10
   0x08049258 <+130>:   jmp    0x804926a <main+148>
   0x0804925a <+132>:   sub    esp,0xc
   0x0804925d <+135>:   push   0x804a02a
   0x08049262 <+140>:   call   0x8049070 <puts@plt>
   0x08049267 <+145>:   add    esp,0x10
   0x0804926a <+148>:   mov    eax,0x0
   0x0804926f <+153>:   lea    esp,[ebp-0x8]
   0x08049272 <+156>:   pop    ecx
   0x08049273 <+157>:   pop    ebx
   0x08049274 <+158>:   pop    ebp
   0x08049275 <+159>:   lea    esp,[ecx-0x4]
   0x08049278 <+162>:   ret
End of assembler dump.
```

위가 main 문의 어셈블리 버전인데, 우리가 주목할 부분은 0x08049222 주소 부분으로 본문에는 화살표로 표시되어 있다. 먼저 atoi를 실행하는데, 이는 문자를 정수로 치환하는 함수이다. 즉 우리가 leviathan6에 1234를 입력하면 이는 문자로 입력되므로 정수로 변환해주는 역할을 한다고보면 무방하다.

![https://user-images.githubusercontent.com/73513005/207078166-ae5ae489-78f4-4247-bf3e-5b128d145eea.png](https://user-images.githubusercontent.com/73513005/207078166-ae5ae489-78f4-4247-bf3e-5b128d145eea.png)

atoi 다음 바로 cmp로 [ebp-0xc]와 eax를 비교하는데, eax의 값은 atoi의 결과 0x4d2이고, 이는 1234를 16진수로 변경한 결과이다. 이제 필요한 건 [ebp-0xc]의 값이다.

![https://user-images.githubusercontent.com/73513005/207078975-0a4a4c57-73e9-4baf-bb02-04ff563afd35.png](https://user-images.githubusercontent.com/73513005/207078975-0a4a4c57-73e9-4baf-bb02-04ff563afd35.png)

값을 알았으니 7123을 입력하여 프로그램을 실행해보자.

![https://user-images.githubusercontent.com/73513005/207079227-da8aeae9-67d3-47ea-99a8-fcd80e3b9b23.png](https://user-images.githubusercontent.com/73513005/207079227-da8aeae9-67d3-47ea-99a8-fcd80e3b9b23.png)

passwd: 8GpZ5f8Hze

## Leviathan 마무리

![https://user-images.githubusercontent.com/73513005/207087888-b444e213-ad20-45ce-aeab-99c0d54dc899.png](https://user-images.githubusercontent.com/73513005/207087888-b444e213-ad20-45ce-aeab-99c0d54dc899.png)

마지막은 훈훈하게 축하 인사로 끝난다. 이 정도 난이도는 정말 최하급이니 리눅스를 처음 해보는 사람이 있다면 한 번 정도 경험해보는 것도 괜찮을 것 같다.