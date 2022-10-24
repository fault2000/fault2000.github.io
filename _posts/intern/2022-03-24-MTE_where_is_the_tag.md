---
layout: post
title: MTE의 메모리에 지정된 태그는 어디에 저장되는가
category: [intern, security]
tags: [mte, arm, memory safety]
fullview: true
comments: true
use_math: true
author: fault2000
---

mte를 둘러보며, 한 가지 궁금한 점이 생겼다.  
mte를 위해선 포인터 앞의 tag가 붙고, 이를 역참조하는 과정에서 tag를 비교해야하므로, 그 메모리에 맞는 tag가 어딘가에는 저장이 되어야할 것이다.  
그러나 이러한 설명이 전에는 찾지 못해 별도로 조사를 해보았다.  

먼저 MTE whitepaper에서는 다음과 같이 언급하고 있다.

```
Memory locations are tagged by adding four bits of metadata to each 16 bytes 
of physical memory. This is the Tag Granule. Tagging memory implements the lock.
Pointers, and therefore virtual addresses, are modified to contain the key
```

각 16 바이트 마다 4bit의 tag를 저장하는 공간을 따로 할당하고, 이를 Tag Granule 이라고 부르는 모양이다.  
고정된 위치가 있는지는 언급이 없는데, 아마 고정된 위치에 태그가 저장되는 순간 조작되기가 쉬워질 것이기에 따로 언급되는 것은 없는 것으로 추정된다. 실제 linux 파일 내의 mte 부분을 나타낸 linux/tools/testing/selftests/arm64/mte 디렉토리의 mte_helper.S 에서도 달리 설명이 있지 않고, 다음과 같이 mte를 위한 새 instruction들을 사용하는 모습들을 보여주고 있다.

```c++
/*
 * mte_insert_random_tag: Insert random tag and might be same as the source tag if
 *                        the source pointer has it.
 * Input:
 *              x0 - source pointer with a tag/no-tag
 * Return:
 *              x0 - pointer with random tag
 */
ENTRY(mte_insert_random_tag)
        irg     x0, x0, xzr
        ret
ENDPROC(mte_insert_random_tag)

/*
 * mte_insert_new_tag: Insert new tag and different from the source tag if
 *                     source pointer has it.
 * Input:
 *              x0 - source pointer with a tag/no-tag
 * Return:
 *              x0 - pointer with random tag
 */
ENTRY(mte_insert_new_tag)
        gmi     x1, x0, xzr
        irg     x0, x0, x1
        ret
ENDPROC(mte_insert_new_tag)

/*
 * mte_get_tag_address: Get the tag from given address.
 * Input:
 *              x0 - source pointer
 * Return:
 *              x0 - pointer with appended tag
 */
ENTRY(mte_get_tag_address)
        ldg     x0, [x0]
        ret
ENDPROC(mte_get_tag_address)

/*
 * mte_set_tag_address_range: Set the tag range from the given address
 * Input:
 *              x0 - source pointer with tag data
 *              x1 - range
 * Return:
 *              none
 */
ENTRY(mte_set_tag_address_range)
        cbz     x1, 2f
1:
        stg     x0, [x0, #0x0]
        add     x0, x0, #MT_GRANULE_SIZE
        sub     x1, x1, #MT_GRANULE_SIZE
        cbnz    x1, 1b
2:
        ret
ENDPROC(mte_set_tag_address_range)
```

보면 irg, gmi, stg 같이 새로 추가된 instruction들을 사용하는 모습을 보여주고 있다.