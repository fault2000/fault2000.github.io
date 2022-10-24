---
layout: post
title: Points-to Analysis
category: [thesis]
tags: [thesis, CFG, security]
fullview: true
comments: true
use_math: true
author: fault2000
---

Points-to Analysis는 compile-time 기술로 포인터 변수와 그 포인터가 프로그램 실행 동안 가리키는 메모리 주소간에 관계를 파악하는데 도움을 준다. 포인터들은 강력한 프로그래밍 구조고, 이들은 포인터 연산과 동적 메모리 할당으로 복잡한 메모리 조작을 프로그램 실행동안 가능하게 한다. 이는 컴파일 시간 때의 포인터 관계를 분석하기 복잡하고 어렵게 한다. 하지만 이는 다시 말해 포인터들이 다양한 다른 컴파일 타임 분석(ex) contant propagation, alias analysis)을 단순화하는데 이점을 제공한다는 뜻도 된다.