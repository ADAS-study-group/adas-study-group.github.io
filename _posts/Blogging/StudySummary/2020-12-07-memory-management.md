---
layout: post
title: Linux Memory Management
author: Jo, SeungHyeon
categories: [Blogging, StudySummary]
tags: [linux, memory]
---
## Memory management technic and virtual memory

- Virtual memory offers maximum sized Virtual Address Space to each Task.

  | CPU   | Size
  | ----- | ---
  | 32bit | 4GB
  | 64bit | 16EB

  - In case of 32bit CPU, Virtual Address Space of a Task does not require the 4GB of Physical memory but takes only as much as the Task uses.
    - More tasks can run with less physical memory
    - No need of memory arrange policy
    - Easy to share or protect memory between tasks
    - Fast task creation

## Physical memory management data structure

- Linux has the information about entire physical memory.
- UMA(Uniform Memory Access): SMP(Symmetric Multi-Processing)
  - Memory and I/O BUS are shared by entire CPUs
  - Possible bottleneck on the resource
- NUMA(Non-Uniform Memory Access)
  - For the sake of the performance, each CPU should access to the nearest memory to fetch data.
- Node
  - Implementation of Bank(Set of memory with the same access speed)
    - Zone structure implemented in "/include/linux/mmzone.h"
    - UMA has one Bank and NUMA has multiple Banks.
  - UMA has one Node
    - The only Node can be accessed with "contig_page_data"
  - NUMA has multiple Nodes.
    - They are managed using list called "pgdat_list"
  - Linux can access the Physical Memory using consistent Node structure no matter what the system is.
    - "pg_data_t" structure is used.
      - "node_present_pages": actual size of the phyiscal memory in the node
      - "node_start_pfn": the index number of the physical memory in the memory map
      - "node_zones": zone structure
      - "nr_zones": the number of zones
    - For the sake of the performance
      - Linux tends to allocate the nearest memory from the CPU working on the Task.
      - Linux tends to reallocate the CPU which have worked on the same task.

    ![Bank-Node]({{ "//assets/img/post/2020-12-07/bank-node.png" | relative_url }})

- Zone
  - Some ISA BUS-based devices are necessary to allocate the region under 16MB of the physical memory.
  - Zones are several regions of the physical memory for the Node.
    - "/include/linux/mmzone.h"
  - The memory in the same zone has the same properties.
  - The memory in the different zone should be managed seperately.

  | Region    | Zone name              | Description
  | --------- | ---------------------- | -----------
  | 0 ~ 16M   | ZONE_DMA or ZONE_DMA32 | saved for some ISA BUS-based devices
  | 16 ~ 896M | ZONE_NORMAL            | mapped from the beginning of the Kernel Space in the Virtual Address Space (e.g. 3072 ~ 3968 M for 32bit)
  | 896 ~ end | ZONE_HIGHMEM           | dynamically allocated as it is needed
  
  - Zone can be the only one in one Node. (e.g. ARM CPU system with 64MB SDRAM)
  - Zone structure has
    - Beginning address and the size of physical memory belong to the Zone
    - free_area structure array for being used by Buddy
    - "watermark" and "vm_stat" determine appropriate memory freeing policy at memory shortage.
      - On the memory shortage, the processes failed to fetch memory are put into "wait_queue" with hashing on "wait_table" variable.

  ```termainl
  $ cat /proc/zoneinfo
  Node 0, zone      DMA
    per-node stats
        nr_inactive_anon 62122
        nr_active_anon 94246
        nr_inactive_file 146827
        nr_active_file 95508
        nr_unevictable 0
        nr_slab_reclaimable 6632
        nr_slab_unreclaimable 8974
        nr_isolated_anon 0
        nr_isolated_file 0
        workingset_refault 0
        workingset_activate 0
        workingset_nodereclaim 0
        nr_anon_pages 92309
        nr_mapped    38734
        nr_file_pages 300632
        nr_dirty     19570
        nr_writeback 0
        nr_writeback_temp 0
        nr_shmem     64526
        nr_shmem_hugepages 0
        nr_shmem_pmdmapped 0
        nr_anon_transparent_hugepages 88
        nr_unstable  0
        nr_vmscan_write 0
        nr_vmscan_immediate_reclaim 0
        nr_dirtied   43945
        nr_written   20874
    pages free     3721
            min      39
            low      48
            high     57
            spanned  4095
            present  3743
            managed  3721
            protection: (0, 3857, 6164, 6164)
        nr_free_pages 3721
        nr_zone_inactive_anon 0
        nr_zone_active_anon 0
        nr_zone_inactive_file 0
        nr_zone_active_file 0
        nr_zone_unevictable 0
        nr_zone_write_pending 0
        nr_mlock     0
        nr_page_table_pages 0
        nr_kernel_stack 0
        nr_bounce    0
        nr_free_cma  0
    pagesets
        cpu: 0
                count: 0
                high:  0
                batch: 1
    vm stats threshold: 8
        cpu: 1
                count: 0
                high:  0
                batch: 1
    vm stats threshold: 8
        cpu: 2
                count: 0
                high:  0
                batch: 1
    vm stats threshold: 8
        cpu: 3
                count: 0
                high:  0
                batch: 1
    vm stats threshold: 8
        cpu: 4
                count: 0
                high:  0
                batch: 1
    vm stats threshold: 8
        cpu: 5
                count: 0
                high:  0
                batch: 1
    vm stats threshold: 8
        cpu: 6
                count: 0
                high:  0
                batch: 1
    vm stats threshold: 8
        cpu: 7
                count: 0
                high:  0
                batch: 1
    vm stats threshold: 8
    node_unreclaimable:  0
    start_pfn:           1
  Node 0, zone    DMA32
    pages free     988431
            min      10549
            low      13186
            high     15823
            spanned  1044480
            present  1011712
            managed  988823
            protection: (0, 0, 2306, 2306)
        nr_free_pages 988431
        nr_zone_inactive_anon 0
        nr_zone_active_anon 0
        nr_zone_inactive_file 0
        nr_zone_active_file 0
        nr_zone_unevictable 0
        nr_zone_write_pending 0
        nr_mlock     0
        nr_page_table_pages 0
        nr_kernel_stack 0
        nr_bounce    0
        nr_free_cma  0
    pagesets
        cpu: 0
                count: 11
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 1
                count: 0
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 2
                count: 0
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 3
                count: 0
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 4
                count: 365
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 5
                count: 0
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 6
                count: 16
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 7
                count: 0
                high:  378
                batch: 63
    vm stats threshold: 48
    node_unreclaimable:  0
    start_pfn:           4096
  Node 0, zone   Normal
    pages free     158188
            min      6306
            low      7882
            high     9458
            spanned  619520
            present  619520
            managed  590396
            protection: (0, 0, 0, 0)
        nr_free_pages 158188
        nr_zone_inactive_anon 62122
        nr_zone_active_anon 94246
        nr_zone_inactive_file 146827
        nr_zone_active_file 95508
        nr_zone_unevictable 0
        nr_zone_write_pending 19570
        nr_mlock     0
        nr_page_table_pages 731
        nr_kernel_stack 7536
        nr_bounce    0
        nr_free_cma  0
    pagesets
        cpu: 0
                count: 373
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 1
                count: 333
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 2
                count: 317
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 3
                count: 282
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 4
                count: 369
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 5
                count: 327
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 6
                count: 268
                high:  378
                batch: 63
    vm stats threshold: 48
        cpu: 7
                count: 123
                high:  378
                batch: 63
    vm stats threshold: 48
    node_unreclaimable:  0
    start_pfn:           1048576
  Node 0, zone  Movable
    pages free     0
            min      0
            low      0
            high     0
            spanned  0
            present  0
            managed  0
            protection: (0, 0, 0, 0)
  ```

- Page frame
  - Managing unit of physical memory by Zone
  - Page structure implemented in "/include/linux/mm_types.h"
  - Pages are supposed to be created for every page frames when the system boots.
  - Pages can be accessed by the global variable called "mem_map"
- Linux's physical memory managing units
  - Physical memory may be composed of one or more Nodes.
  - Node may be composed of one or more Zones.
  - Zone may be composed of many Page frames.

  ![Node-Zone]({{ "//assets/img/post/2020-12-07/node-zone.png" | relative_url }})

## Buddy and Slab

- Linux allocates physical memory to tasks by the "Page frame" unit.
  - At least 4KB, which can be changed to be 8KB, 2MB, etc.
  - External Fragmentation: When task requests bigger size than several page frames and the residual is smaller than one page frame.
  - Internal Fragmentation: When task requests smaller size than one page frame.

- Buddy Allocator
  - External Fragmentation
  - Implemented through the free_area structure array in the Zone structure (one Buddy for one Zone)
    - free_area structure has
      - free_list
      - map

      ![free_area]({{ "//assets/img/post/2020-12-07/free_area.png" | relative_url }})

      - The number of free_area will be the number of squares of 2 which calculates the maximum number of page frames for one buddy. (e.g. 4KB, 8KB, 16KB, ..., 4MB)
  - Example
    - On 2 pages are requested

      ![Buddy allocator procedure 1]({{ "//assets/img/post/2020-12-07/buddy-allocator1.png" | relative_url }})

    - On another 2 pages are requested

      ![Buddy allocator procedure 2]({{ "//assets/img/post/2020-12-07/buddy-allocator2.png" | relative_url }})

    - On page 11 are freed

      ![Buddy allocator procedure 3]({{ "//assets/img/post/2020-12-07/buddy-allocator3.png" | relative_url }})

  - Lazy Buddy

    ![Lazy Buddy]({{ "//assets/img/post/2020-12-07/lazy-buddy.png" | relative_url }})

    - "free_area::map" -> "free_area::nr_free": number of free Page frames

    ```terminal
    $ cat /proc/buddyinfo
    Node 0, zone      DMA      1      0      0      1      2      1      1      0      0      1      3
    Node 0, zone    DMA32      3      2      4      3      6      4      4      4      3      1    963
    Node 0, zone   Normal     54    244    185    109     41     22      7      3      2      9    145
    ```

- Slab Allocator
  - Internal Fragmentation