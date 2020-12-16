---
author: devlee8585
title: Memory Management(buddy,slab)
categories: [Blogging, Tech, OS, Linux, Kernel]
tags: [linux, kernel, memory, buddy, slab]
---
## 메모리 관리
메모리 관리 기법중 Buddy와 Slab 메모리 할당 기법을 공부해보자

### Intro
물리 메모리는 페이지 프레임 단위인 4KB씩 할당하게 된다(8KB, 2MB로 설정 변경 가능) 
4KB의 프레임 바이트를 쓸대 1Byte 나 16KB 물리 메모리를 공간을 요청하면 내부/외부 단편화가 발생된다
이런 문제를 어떻게 해결하여 할당하고 해제하는지 알아보자

### Technical Terms
- 외부단편화 (External fragmentation)
프로그램을 할당하고 난 다음 아주 작은 크기로 남은 조각들이 생겨 사용할 수 없는 작은 공간들이 많이 생길 수 있습니다.
이 공간들을 합치면 요구되는 공간을 할달할 수 있음에도 불구하고 연속적인 공간이 아니라서 할당하지 못하는 상황을 외부 단편화라고 합니다.
ex) A,B,C 사이의 8K공간이 남아있음에도 7K의를 할당할 수 없는 문제 발생

![Desktop View]({{ "/assets/img/post/2020-12-12/04.external-fragmentation.png" | relative_url }})

- 내부단편화 (Internal fragmentation)
할당되는 최소 블록 크기보다 적게 할당하여 내부영역을 낭비하는 경우를 내부 단편화라고 합니다. 
ex) 할당하는 최소 블록 크기를 10K라고 가정하고
만약 7K를 할당하는 경우 나머지 3K를 낭비하게 된다.

![Desktop View]({{ "/assets/img/post/2020-12-12/05.internal-fragmentation.png" | relative_url }})


### Detailed description

![Desktop View]({{ "/assets/img/post/2020-12-12/06.zone-freearea.png" | relative_url }})

```
struct free_area{
  struct list_head free_list[MIGRATE_TYPES]; //linked list
  unsigned long nr_free; //free한 page의 개수
}
```
### Buddy / Slab
#### 버디 할당자 (Buddy Allocator)
- 4KB보다 큰 크기 메모리공간을 요청했을 경우 
  => 16KB를 할당해주는 버디 할당자(Buddy Allocator) 사용 
   buddy는 연속적인 page 관리를 위한 것
  관리 부하가 적고, 외부 단편화(External Fragmentation) 줄일 수 있는 장점 있음

- order는 개의 연속적인 page 가 한묶음(?)인지 나타태는 숫자ㅊ
  == free_area[]
  ex 1) order 가 1이라면 2^1 으로 2개의 page 가 하나의 block
  ex 2) order 가 10이라면 2^10으로 1024개의 page들이(4MB) 하나의 묶음
  ex 3)시스템이 16KB 의 메모리를 요청했다면 2^4(16), 즉 order 4인 free_area[4].free_list 에 연결되어 있는 page 를 받게 된다는 것

![Desktop View]({{ "/assets/img/post/2020-12-12/07.order-freearea-pageblock.png" | relative_url }})

##### 요청되는 order 에 연속적인 page 를 할당해줄 수 있는 것이다. 물론 꼭 연속적이지 않더라고 할당은 할 수 있는데 최대한 연속적인 메모리를 할당해 주려고 노력하기 위한 알고리즘인 것이다.

![Desktop View]({{ "/assets/img/post/2020-12-12/08.buddy-1.png" | relative_url }})

* 2page 요청 시 
  1. 현재 5,10,12,13,14,15 페이지가 비어있다
  2. 2page가 요청되면 order 1번(2^1=2page)에서 찾는다.(2^order 의 개수로 free_list 에서 관리)
  3. order 1번에 없으면 상위 order인 2번에서 확인한다.
  4. order 2번에서 12번 page부터 4개가 free하다고 알려준다
  5. 12번을 order 1번 2개로 쪼갠다
  6. 요청된 2개 page를 12번부터 할당한다.
  7. 남은 14,15번은 order의 1번에 매달린다.
   
* 해제 시
  1. 0번 page 해제 요청이 온다.
  2. 0번이 해제되고 order 0 의 free_list 로 가서 붙는다.
  3. 이번엔 1번의 page 가 해제 요청이 들어온다.
  4. 1번이 해제되고 order 0의 free_list 로 가서 붙는다.
  5. 해제하고 나서 buddy를 찾는다. 바로 0번 page 가 buddy가 되고 두 녀석이 합쳐서 상위 order 로 가서 붙게 된다. 

  * 결과적으로 order 0에 있던 0,1 번 page는 없어지고, order 1에 0번의 page가 붙게 되는 것이다.

- order에 등록/해제하는 것은 해당 비트맵으로 판단 한다.
#### Lazy Buddy
- 커널 버전 2.6.19부터는 free area의 구조와 버디 할당자의 구현이 조금 바뀌었다.
- free_area 구조체의 map포인터를 **nr_free변수로 바꾼다.
  **nr_free : zone내에서 비시용중인 페이지 프레임의 개수이다
- 기존 버디 문제점 : 페이지 프레임을 할당해주기 위해서는 큰 페이지를 쪼개서(그러기 위해선 비트맵 역시 수정되어야 한다) 할딩해줘야 한다- 그런 뒤 해제된다면? 다시 큰 페이지로 합쳐서(역시 비트맴 수정이 필요하다) 관리해야 한디. 이런 작업이 반복된다면 할당/해제를 위해 많은 **오버헤드가 동반된다
** 오버헤드(Overhead)란 어떤 처리를 하기 위해 들어가는 간접적인 처리 시간 · 메모리 등을 말한다.
- 할당되었던 페이지프레임을 구태여 합치치 말고, 합치는 작업을 뒤로 **미루는 것 이것이 Lazy버디 등장 배경이다.
** 미룬다는 것은 zone에 가용 메모리가 부족한 경우 해제된 페이지의 병합을 하겠다는 것이다.
- __free_pages(){__free_one_page()} 에서 해제하는 페이지가 MAX_ORDER만큼 루프를 돌면서 해제하는 페이지가 버디와 합쳐져서 상위 order에서 관린될 수 있는지 파악하고 가능한 경우 order의 nr_free를 감소시키고 상위로 페이지를 이동 시킨 뒤 상위 order의 nr_free를 증가시킨다.
- 페이지 할당 함수 __alloc_pages()
- 페이지 해제 함수 __free_pages()
- 시스댐의 버디 할당자 관련 정보는 “cat /proc/buddyinfo" 명령을 통해 확인해 볼 수 있다.
![Desktop View]({{ "/assets/img/post/2020-12-12/09.buddy-info.png" | relative_url }})

#### 슬랩 할당자(Slab Allocator)
- 4KB보다 작은 크기 요청했을 경우 
  => 슬랩 확장자(Slab Allocator) 사용 
  30byte 요청했을 때 4KB를 할당해주면 내부 단편화(Internal Fragmentation) 문제 발생할 수 있는데 슬랩 확장자로 이를 해결

- 미리 4KB 페이지 프레임을 한 개 할당받은 뒤， 이 공간을 64Byte 크기 64개로 분할해 둔다.(64 x 64=4096(4KB))
- 마치 일종의 cache로 사용하는 것이고 cache의 집합(?)을 통해 메모리를 관린하는 정책을 슬랩할당자라고 부른다.
![Desktop View]({{ "/assets/img/post/2020-12-12/10.slab.png" | relative_url }})

- Object (64byte) --> slab (**full/free/partial) --> cache
  **캐시에는 다음과 같은 3가지의 슬랩 리스트가 있다.
  slab_full : 슬랩내의 오브젝트가 전부 사용 중인 것
  slab_partial : 슬랩내의 오브젝트가 사용중인 것과 미사용중인 것이 혼재되어 있는 것
  slabs_empy : 슬랩내의 오브젝트가 전부 미사용인 슬랩 리스트

- 캐시는 관리가 필요한 오브젝트 종류별로(예를 들면task_struct, file, buffer_head, inode 등) 작성되고 그 오브젝트의 슬랩을 관리한다.
- 슬랩은 하나 이상의 연속된 물리 페이지로 구성되어 있으며 일반적으로 하나의 페이지로 구성된다. 
- 캐시는 이러한 슬랩들의 복수개로 구성된다.
- kmalloc이 어떻게 슬랩 할당자를 사용할까?
  => 초기화 될 때 kmem_cache_init 함수를 통하여 자주 사용되는 커널의 오브젝트들의 크기를 고려하여 일반적으로 사용할 목적으로 추가적인 캐시들을 생성

참고사이트
https://sycho-lego.tistory.com/
https://jiming.tistory.com/131
https://neohtux.tistory.com/14?category=620069