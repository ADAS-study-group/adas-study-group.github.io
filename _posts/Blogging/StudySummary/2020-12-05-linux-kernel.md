---
layout: post
title: Linux Kernel Structure
author: Jo, SeungHyeon
categories: [Blogging, StudySummary]
tags: [linux, kernel, structure]
---
## Linux kernel structure

- Operating System is a resource manager
- Resource

  Physical resource | Abstract resource
  ----------------- | ------------------
  CPU               | Task
  Memory            | Segment, Page
  Disk              | File
  Network           | Communication protocol, Packet
  
  - Abstract resource only
    - Security
    - Access control

- Linux kernel's conceptual structure

  ![Linux kernel conceptual structure]({{ "//assets/img/post/2020-12-05/linux-conceptual-structure.png" | relative_url }})

  Linux kernel offers services in the user space with system calls through the system call interfaces.
  - Linux Kernel (Conceptual Structure)
    - Task Manager
      - Task creation
      - Task execution
      - State transition
      - Scheduling
      - Signal Handling
      - Inter Process Communication
    - Memory Manager
      - Physical memory management
      - Virtual memory management
      - Segmentation
      - Paging
      - Page fault handling
    - File System
      - File creation
      - Access control
      - inode management
      - Directory management
      - Super block management
    - Network Manager
      - Socket interface
      - TCP/IP communication protocol
    - Device Drive Manager
      - Manage device drivers for Disk, Terminal, CD, Network card, etc.

- Linux kernel source level structure
  - Source tree under "/usr/src/kernels/" (branch "linux-4.19.y")

    ![Linux kernel source tree linux-4.19.y]({{ "//assets/img/post/2020-12-05/linux-kernel-source-4.19.y_root.png" | relative_url }})

    - /kernel directory
      - Implementation of the Task manager
      - HW dependent task managing logics are in /arch/$(ARCH)/kernel.
        - $(ARCH) is i386, arm, etc.
    - /arch directory
      - Acronym of "architecture"
      - /arch/$(ARCH)/kernel
        - $(ARCH) is i386(Intel), arm (Advanced RISC Machine), 68 series(Motorolla), Sparc(SUN), PPC(Power PC, IBM) etc.
          - /arch/x86

          Source Directory   | Description
          ------------------ | -----------
          /arch/x86/boot     | bootstrap code for system booting
          /arch/x86/kernel   | HW dependent task manage code like context switch
          /arch/x86/mm       | HW dependent memory manage code like page fault handling
          /arch/x86/lib      | library function code used by the kernel
          /arch/x86/math-emu | FPU(Floating Point Unit) emulator

    - /fs directory
      - System call implementation like open(), read(), write()
      - At least 60 different file systems are available for Linux
        - Representative file systems are ext2, ext3, ext4, nfs, fat, proc, sysfs, devfs, isofs, ntfs, reiserfs, f2fs, xfs, etc.
      - Virtual File System for offering consistent interface for user
    - /mm directory
      - Implementation of the Memory manager
      - Physical memory management
      - Virtual memory management
      - Memory object allocated for each task management
    - /driver directory
      - Implementation of the Device manager
      - Management and virtualization of Disk, Terminal, Network card, etc.
      - 3 different device drivers
        - Block Device Driver accessed by the file system
        - Character Device Driver accessed by user application program through its device file
        - Network Device Driver accessed by TCP/IP
      - Others: 3 categories are not enough because of many kinds of different devices like USB, LCD, DSP, Sound, etc.
    - /net directory
      - Decent portion of the whole linux kernel source amount
      - Many different network protocols are available like TCP/IP, UNIX domain communication protocol, 802.11, IPX, RPC, AppleTalk, bluetooth, etc.
      - Implementation of Abstract layer of many different communication protocol
      - Implementation of Socket offering user interface
    - /ipc directory
      - Implementation of Inter-Process Communication supported by linux kernel
      - Representative IPCs are PIPE(/fs), Signal(/kernel), SYS V IPC, Socket(/net), etc.
      - Implementation of message passing, shared memory, semaphore, etc.
    - /init directory
      - Implementation of main start function to initialize the kernel
      - /arch/$(ARCH)/kernel:
        - HW dependent initialization at "head.S" and "mics.c"
        - Global kernel initialization at start_kernel()
    - /include directory
      - Linux kernel headers at /include/linux
      - HW dependent headers at /include/asm-$(ARCH)
    - other directory
      - /Documentation directory for descriptions
      - /lib directory for implementation of kernel library functions
      - /scripts directory for scripts used when configure and compile the kernel

    Physical resource | Abstract resource              | Conceptual Manager | Source directory
    ----------------- | ------------------------------ | ------------------ | ---------
    CPU               | Task                           | Task Manager       | /kernel
    Memory            | Segment, Page                  | Memory Manager     | /mm
    Disk              | File                           | File System        | /fs
    Network           | Communication protocol, Packet | Network Manager    | /ipc
    Devices           | Device drivers for each devices| Device Manager     | /driver

## Linux kernel compile

- For new version of linux kernel, just compile and reboot the new kernel.

  |              | hello.out   | kernel
  | ------------ | ----------- | ------
  | executable   | hello.out   | bzImage or zImage
  | stored at    | file system | memory
  | access level | user level  | kernel level
  | compile      | gcc         | gcc with make file

- 3 steps for making linux kernel
  1. Kernel configuration
  2. Kernel compile
  3. Kernel installation
  