---
author: devlee8585
title: Memory Management(node,zone,pageframe)
categories: [Blogging, Tech, OS, Linux, Kernel]
tags: [linux, kernel, memory, node, zone, page frame,]
---
## 메모리 관리
page frame, zone, node구조를 통한 전체 물리 메모리 관리 기법을 살펴보자

### Detailed description
#### SMP(Symmetric Multiprocessing)
- 복수개의 CPU를 가지고 있는 컴퓨터 시스템 중 모든 CPU가 메모리와 출력 버스 등을 공유하는 구조
  
#### UMA(Uniform Memory Access)
- 모든 프로세서 들이 상호간에 연결되어서 하나에 메모리를 공유하는 기술
- 프로세서들은 메모리의 어느 영역이던지 접근 가능하나 병목 현상이 생길 수 있다.

#### NUMA(Non-Unifom Memory Access)
- CPU들을 몇개의 그룹으로 나누고 각각의 그룹에게 별도의 지역 메모리를 주는 기술

#### 뱅크(Bank) 
- 접근 속도가 같은 메모리의 집합
- UMA구조는 한개의 뱅크 존재
- NUMA구조는 복수개의 뱅크 존재
- 한개의 램(RAM)(?)

#### 노드(Node)
- 뱅크를 표현하는 자료구조(~/include/linux/mmzone.h)
- 램관련 자료구조(?
- UMA구조에서는 한 개의 노드를 사용하기 때문에 전역변수인 contig_page_data (linux/mm/page_alloc.c)를 통해 접근이 가능하고, NUMA구조라면 복수개의 노드를 pgdat_list (linux/arch/ia64/mm/discontig)라는 리스트를 통해 관리하여 하드웨어의 시스템에 관계없이 노드라는 일관된 자료구조를 통해서 전체 물리메모리에 접근할 수 있다
![Desktop View]({{ "/assets/img/post/2020-12-12/01.band-node-UMA-NUMA.png" | relative_url }})

##### ! 리눅스는 하드웨어System에 관계없이 node라는 일관된 자료 구조로 전체 physical memory(물리메모리)에 접근할 수 있게 된다.

#### 존(Zone)
- node의 일부분을 따로 관리 할 수 있게 만든 자료구조 메모리와는 별개로 따로 관리 되어야하는 메모리의 집합이다(~/include/linux/mmzone.h)
-노드와 같은 파일 사용-
- 일부 ISA 버스 기반의 디바이스 지원을 위해, Node의 일부분(16MB이하 부분)을 따로 관리 할 수 있도록 자료 구조를 만듦
- x86 구조에서는 
  ZONE_DMA : 0~16MB
  ZONE_NORMAL : 16 ~ 896M
  ZONE_HIGHMEM : 896~end
  구조를 가진다. 

-> ZONE_HIGHMEM 이 있는 이유
  : 물리 메모리가 4GB 라고 하더라도, 실제 커널의 가상 주소 공간은 1GB 이므로, 나머지 3GB 가 User Process 들에 의해 쓰이지 않더라도, 커널은 그 공간을 쓸 수가 없다.  이러한 비효율성을 극복하기 위해서, ZONE_NORMAL 영역(~896MB) 까지는 커널 주소공간과 물리 메모리 공간을 1:1 매핑하고,, 나머지 부분에 대해서는 필요할 때 동적으로 커널 주소공간과 연결시켜준다. 

![Desktop View]({{ "/assets/img/post/2020-12-12/02.Zone.png" | relative_url }})

```
cat /proc/zoneinfo  (zone 정보 분석을 위해 사용)

Node 0, zone   Normal
  pages free     20316  // free page 의 수. 즉, 사용 가능한 물리 Page 의 수를 말한다.
        min      10781
        low      13476
        high     16171
// 만약 프로세스가 zone 에 메모리 할당 요청했는데, free 페이지 부족하다면, 이러한 변수가 wait_queue 에 저장되며,  메모리 해제 정책에 따라 메모리 해제 후, wait_queue 에 있는 프로세스에게 메모리 할당해준다.

        scanned  0
        spanned  224254  
        present  224254  // 현재 Zone 에 존재하는 물리 Page 의 총 갯수
        managed  217240
    nr_free_pages 20316
    nr_alloc_batch 1571
    nr_inactive_anon 3110
    nr_active_anon 2976
    nr_inactive_file 77582
    nr_active_file 83926
    nr_unevictable 0
    nr_mlock     0
    nr_anon_pages 5671
    nr_mapped    2668
    nr_file_pages 161923
    nr_dirty     5
    nr_writeback 0
    nr_slab_reclaimable 18934
    nr_slab_unreclaimable 2635
    nr_page_table_pages 126
    nr_kernel_stack 196
    nr_unstable  0
    nr_bounce    0
    nr_vmscan_write 16
    nr_vmscan_immediate_reclaim 0
    nr_writeback_temp 0
    nr_isolated_anon 0
    nr_isolated_file 0
    nr_shmem     412
    nr_dirtied   293933
    nr_written   294944
    nr_anon_transparent_hugepages 3
    nr_free_cma  0
        protection: (0, 0, 5279, 5279)
  pagesets
    cpu: 0
              count: 141
              high:  186
              batch: 31
  vm stats threshold: 8
  all_unreclaimable: 0
  start_pfn:         4096  // 물리 메모리의 시작 page frame number.  PageSize=4kb 인 경우 왼쪽으로 12bit shift 하면 물리 메모리의 시작주소이다.  해석해보면 Node0-ZONE_NORMAL 의 물리 메모리 시작주소는 0x1000000 이 된다.

  inactive_ratio:    1
```

#### Page Frame
- 각각의 zone 은 자신에 속해 있는 물리 메모리들을 관리하는데, 이 물리 페이지의 최소 단위를 페이지 프레임(page frame) 이라고 부른다. 
- 각 페이지 프레임은 page  구조체에 의해 관리된다. 
  (include/linux/mm_types.h)
- 리눅스는 모든 물리 메모리에 접근 가능해야 하므로, 시스템 부팅 시 모든 page frame 에 대한  page 구조체가 할당되어 메모리에 저장된다.   mem_map 이라는 전역 배열을 통해 접근 가능하다.
- 이러한 page frame 은 Node (pg_data_t 구조체) 안에 node_mem_map 변수를 통해 연결된다.
![Desktop View]({{ "/assets/img/post/2020-12-12/03.page-frame.png" | relative_url }})


### 정리
-> page frame : 페이지 최소단위, 하나의 페이지로 관리
-> zone : 복수개의 페이지 프레임으로 구성
-> node : 하나 또는 복수개의 Zone으로 구성
-> 리눅스 전체 물리 메모리 : 하나 또는 복수개의 node로 구성

참고사이트
https://jiming.tistory.com/131
https://neohtux.tistory.com/14?category=620069