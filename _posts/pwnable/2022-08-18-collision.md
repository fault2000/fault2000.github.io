---
layout: post
title: pwnable collision
categories: [pwnable]
tags: [pwnable, Toddler's Bottle]
fullview: true
comments: true
use_math: true
author: fault2000
---

![image](https://user-images.githubusercontent.com/73513005/185577148-55a00ca7-06e9-4e78-b555-a30fbb5f8173.png)

처음 접속하면 전 문제와 같은 col, col.c, flag가 있다. 당연하게도 flag는 우리가 볼 수 없으므로, 일단 col을 무작정 실행해보자.  

![image](https://user-images.githubusercontent.com/73513005/185577204-693639da-abb0-4457-a198-30d47b332616.png)

이렇게 친절할 수가. 틀린점을 적시해준다. 그럼 저 양식에 맞춰 다시 넣어보자.  

![image](https://user-images.githubusercontent.com/73513005/185577271-d54479bb-7aa6-4248-a96c-37398cdf9314.png)

총 20 바이트를 넣어주라고 되어있다. 그럼 20바이트 숫자 아무거나 넣어보자. 숫자 하나당 1바이트임을 기억하자.  

![image](https://user-images.githubusercontent.com/73513005/185577353-b9c29b2b-45f2-4274-ac21-5efbc02c385b.png)

틀렸단다. 이렇게 아까울 수가. 농담은 여기까지하고 이제 col.c 코드를 보도록하자.  

***

![image](https://user-images.githubusercontent.com/73513005/185577444-ee9d8b9b-f5c0-4187-8294-3f6b2e2a224f.png)

col.c는 크게 메인 함수와 check_password 함수 두 개로 이루어져있다. 일단 메인부터 보자.  

![image](https://user-images.githubusercontent.com/73513005/185577486-cca117a3-f1d7-484d-a0e8-f941e42bbf92.png)

일단 위 2 if문은 우리가 앞에서 실험해보았던 오류 2개를 걸러내는 과정을 담고 있다. 먼저 첫번째 조건은 인자가 1개 이상 있어야하고, 그 다음 조건은 첫 인자가 20바이트의 크기를 가져야한다. 이 두 과정을 통과했다면 본격적으로 검사를 시작한다.  
먼저 hashcode인 0x21DD09EC가 있다. 이 수와 check_password에 첫 인자를 준 리턴값이 같다면, 우리는 flag를 볼 수 있다. 그렇다면 이제 check_password를 볼 시간이다.  

![image](https://user-images.githubusercontent.com/73513005/185577578-0db42535-7b65-417e-b20a-f655b15a574c.png)

먼저 char\*인 매개변수 p를 int\*로 변환한다. 그 다음 정수가 된 변수는 각 크기가 4바이트인데, 총 크기 20바이트를 4바이트씩 5개로 나누고, 그 값을 각각 더한다. 그 더한 수를 리턴하면서 끝이 나게 되는 함수이다.  
여기까지 보면 단순하게 문제가 해결된다. 우리의 값인 hashcode를 5로 나누고, 그 값을 반복해서 4바이트 안에 넣어주면 끝이다.  

***

이제 실질적인 문제 풀이를 해보자. 먼저 0x21DD09EC를 5로 나누면 0x6c5cec8과 4의 나머지가 생긴다. 아무렇게나 넣으면 되니 여기선 0x6c5cec8 4개와 0x6c5cecb 하나로 구성되는 수를 만들어야한다. 당연히 십진수로 하면 20바이트가 초과될 것이 뻔하니, 리틀 엔디안 구조로 16진수를 이용해 답을 구성하면 대략 \xc8\xce\xc5\x06 x 4 + \xcb\xce\xc5\x06이 답이 될 것이다.  

그럼 이제 답을 적어내보자.  

![image](https://user-images.githubusercontent.com/73513005/185577844-307946da-b606-4015-8955-53299b46da94.png)

그럼 답이 도출됨을 볼 수 있다.