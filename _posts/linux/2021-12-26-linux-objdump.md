---
layout: post
title: 리눅스 objdump 바이너리 유틸리티
categories: [linux]
tags: [raspberryPi, linux]
fullview: true
comments: true
author: fault2000
---

**바이너리 유틸리티**는 오브젝트 포맷의 파일을 조작할 수 있는 프로그램이다.
이들 중 우리가 사용할 objdump는 라이브러리 or ELF 형식의 파일을 어셈블리어로 출력한다.
사용방식은 다음과 같다. **objdump <option(s)> <file(s)> Display**
objdump의 옵션으로 -x, -d가 각각 있으며 -x는 모든 헤더를, -d는 실행 가능 영역의 모든 어셈블러 내용을 보여준다.

이 기능을 활용해 다음장에서는 인터럽트, 커널 스케줄링, 시스템 콜 등의 요소를 분석할 예정이다.