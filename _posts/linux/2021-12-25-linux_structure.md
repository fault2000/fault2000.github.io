---
layout: post
title: 리눅스 커널 소스 구조
category: [linux]
tags: [raspberryPi, linux]
fullview: true
comments: true
author: fault2000
---

리눅스의 처음 모습을 보면 상당히 난해하다. 처음 소스를 본다면 직관적인 driver, documenttation 등의 이름이 눈에 띄지만 그뿐, 결국 각 디렉토리의 의미를 정확하게 알기란 힘들다.
책에서는 각 디렉토리의 구조를 다음과 같이 설명하고 있다.
<h4>arch</h4>
아키텍쳐별 커널 코드
<h4>include</h4>
커널 코드 빌드에 필요한 헤더 파일
<h4>Documentation</h4>
커널 기술 문서가 있는 폴더, 기본 동작을 설명하는 문서 찾을 수 있음
<h4>kernel</h4>
커널 핵심 코드, 하위 디렉토리는 다음과 같다.
<h5>irq: 인터럽트 관련 코드</h5>
<h5>sched: 스케줄링 코드</h5>
<h5>power: 커널 파워 관리 코드</h5>
<h5>locking: 커널 동기화 관련 코드</h5>
<h5>printk: 커널 콘솔 관련 코드</h5>
<h5>trace: frace 관련 코드</h5>
<h4>mm(Memory Management)</h4>
가상 메모리 및 페이징 관련 코드, arch/*/mm
<h4>drivers</h4>
모든 시스템의 디바이스 드라이버 코드, 하부 디렉터리에 드라이버 종류별 소스
<h4>fs</h4>
모든 파일 시스템 코드
<h4>lib</h4>
커널에서 제공하는 라이브러리 코드, arch/*/lib
