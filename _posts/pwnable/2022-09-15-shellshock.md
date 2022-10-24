---
layout: post
title: pwnable shellshock
category: [pwnable, security]
tags: [pwnable, Toddler's Bottle]
fullview: true
comments: true
use_math: true
author: fault2000
---

shellshock는 CVE-2014-6271의 번호를 부여받은 취약점으로, 환경 변수를 통해 공격자가 명령어를 실행할 수 있는 취약점이다.  
이는 특정 bash 버전의 환경 변수와 함수 선언에서 나타나는 버그를 이용해 명령어를 실행시킨다.  

<img width="418" alt="image" src="https://user-images.githubusercontent.com/73513005/193805711-795ccd58-22c6-40a3-aedc-bcdc2db15d80.png">

일단 접속하여 내용을 보면, shellshock와 bash, flag를 확인할 수 있고, shellshock 내용은 단순히 설명해 getegid() 함수를 통해 현 process의 effective 그룹 id를 얻고, 이를 인자로 setresuid, setresgid 함수를 호출한다. 이는 프로세스의 권한을 그룹 id로 만든다고 생각하면 편하다.  

중요한 점은 다음 부분으로, system 함수에서 현재 위치에 있는 ./bash를 실행하고, 거기서 shock_me를 출력한다.  

문제의 이름부터 그렇듯 shellshock 문제일테니 우리에게 주어진 bash는 shellshock 취약점을 담고 있을 것이다. 한번 테스트해보자.  

<img width="353" alt="image" src="https://user-images.githubusercontent.com/73513005/193819254-83c55162-719c-4d36-b7f6-396cb188de99.png">

일단 우리는 환경변수를 만들어주어야한다. 여기서 선언되는 형태는 함수를 선언한다. 원래라면 여기서 끝나야 하지만, shellshock는 전혀 예상하지 못한 취약점이다.  

<img width="507" alt="image" src="https://user-images.githubusercontent.com/73513005/193821629-b93e7071-d63c-43cf-a1a3-f05c216ffe97.png">

우리가 정상적으로 선언한 함수 환경변수 뒤에 새로운 명령들이 추가되었다. 그리고 bash 쉘을 실행하니 이 뒤에 있던 명령들이 실행됨을 볼 수 있다.  

bash 명령을 사용하면 환경변수가 초기화되는데, 이때 함수의 모양을 하던 a가 버그로 진짜 함수로 변경된다. 또한 뒷부분 명령은 원래라면 무시되거나 없어져야하지만, 마찬가지로 버그로 인해 방금처럼 함수 선언으로 끝나는 것이 아닌 실행까지 되버린다.  

이런 문제를 알고만 있다면 이번 문제는 금방 해결될 수 있다. 그룹 id로 명령어를 실행하는 shellshock가 bash를 실행하는 것을 우린 안다. 그렇다면 적당한 함수 하나 실행 후 뒤에 flag를 보는 명령어만 추가해주면 간단히 풀린다.  

<img width="436" alt="image" src="https://user-images.githubusercontent.com/73513005/193827138-b45ea10b-02c0-472b-8cbd-029c314ae326.png">

