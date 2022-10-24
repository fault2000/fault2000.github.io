---
layout: post
title: MTE kernel assembly
category: [intern]
tags: [mte, arm, memory safety]
fullview: true
comments: true
author: fault2000
use_math: true
---

커널 속에서 MTE가 사용되는지 확인하기 위해, 커널을 직접 살펴본 결과, 어느 정도의 결론에 도달할 수 있었다.   
MTE에 사용되는 특수한 명령어들이 사용되는지 확인하는 과정을 거쳤다. 사용되는 명령어는 다음 그림에 있다.  

![image](https://user-images.githubusercontent.com/73513005/158526078-e9aa69b1-8df1-4ca3-b262-331507706585.png)  

위 명령어를 검색한 결과, IRG, CMPP같은 새로운 태그를 넣거나, 태그를 확인하는 과정은 전혀 발견되지 않았다.  
유일하게 발견된 명령어는 STG, LDG 같은 명령어 혹은 파생형들과 MTE를 활성화하는 과정인 set_tagged_addr_ctrl과 get_tagged_addr_ctrl인데, 이들은 tag가 있는 페이지를 복사하거나, tag를 제거하거나, 복구하는 함수(mte_copy_page_tags, mte_clear_page_tags 등)에서 사용되므로 사실상 커널 자체적으로 사용되지 않음을 확인할 수 있었다.  

![image](https://user-images.githubusercontent.com/73513005/170437635-58cea146-fb06-4e04-a00f-ad211b085cd7.png)

![image](https://user-images.githubusercontent.com/73513005/170437709-74560d8f-293c-4de2-b296-8e595cd83df5.png)

![image](https://user-images.githubusercontent.com/73513005/170437776-d332c9a1-7f34-4484-ad51-68acbcb6abf9.png)