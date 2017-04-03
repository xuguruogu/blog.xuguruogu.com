---
title: /proc/buddyinfo理解
date: 2017-04-03 09:43:04
tags: [linux, memory]
---

/proc/buddyinfo
This file is used primarily for diagnosing memory fragmentation issues. Using the buddy algorithm, each column represents the number of pages of a certain order (a certain size) that are available at any given time. For example, for zone DMA (direct memory access), there are 90 of 2^(0*PAGE_SIZE) chunks of memory. Similarly, there are 6 of 2^(1*PAGE_SIZE) chunks, and 2 of 2^(2*PAGE_SIZE) chunks of memory available.

The DMA row references the first 16 MB on a system, the HighMem row references all memory greater than 4 GB on a system, and the Normal row references all memory in between.

Each column of numbers represents the number of pages of that order which are available. In the example below, there are 7 chunks of 2 ^ 0 * PAGE_SIZE available in ZONE_DMA, and 12 chunks of 2 ^ 3 * PAGE_SIZE available in ZONE_NORMAL, etc...

This information can give you a good idea about how fragmented memory is and give you a clue as to how big of an area you can safely allocate.

When a Linux system has been running for a while memory fragmentation can increase which depends heavily on the nature of the applications that are running on it. The more processes allocate and free memory, the quicker memory becomes fragmented. And the kernel may not always be able to defragment enough memory for a requested size on time. If that happens, applications may not be able to allocate larger contiguous chunks of memory even though there is enough free memory available. Starting with the 2.6 kernel, i.e. RHEL4 and SLES9, memory management has improved tremendously and memory fragmentation has become less of an issue.

To see memory fragmentation you can use the magic SysRq key. Simply execute the following command:
# echo m > /proc/sysrq-triggerThis command will dump current memory information to /var/log/messages. Here is an example of a RHEL3 32-bit system:

Jul 23 20:19:30 localhost kernel: 0*4kB 0*8kB 0*16kB 1*32kB 0*64kB 1*128kB 1*256kB 1*512kB 1*1024kB 0*2048kB 0*4096kB = 1952kB)
Jul 23 20:19:30 localhost kernel: 1395*4kB 355*8kB 209*16kB 15*32kB 0*64kB 0*128kB 0*256kB 0*512kB 0*1024kB 0*2048kB 0*4096kB = 12244kB)
Jul 23 20:19:31 localhost kernel: 1479*4kB 673*8kB 205*16kB 73*32kB 21*64kB 847*128kB 473*256kB 92*512kB 164*1024kB 64*2048kB 28*4096kB = 708564kB)The first line shows DMA memory fragmentation. The second line shows Low Memory fragmentation and the third line shows High Memory fragmentation. The output shows memory fragmentation in the Low Memory area. But there are many large memory chunks available in the High Memory area, e.g. 28 4MB.

If memory information was not dumped to /var/log/messages, then SysRq was not enabled. You can enable SysRq by setting sysrq to 1:
# echo 1 > /proc/sys/kernel/sysrqStarting with the 2.6 kernel, i.e. RHEL4 and SLES9, you don’t need SysRq to dump memory information. You can simply check /proc/buddyinfo for memory fragmentation.

Here is the output of a 64-bit server running the 2.6 kernel:
# cat /proc/buddyinfo
    Node 0, zone DMA 5 4 3 4 3 2 1 0 1 1 2
    Node 0, zone Normal 1046 527 128 36 17 5 26 40 13 16 94
# echo m > /proc/sysrq-trigger
# grep Normal /var/log/messages | tail -1
    Jul 23 21:42:26 localhost kernel: Normal: 1046*4kB 529*8kB 129*16kB 36*32kB 17*64kB 5*128kB 26*256kB 40*512kB 13*1024kB 16*2048kB 94*4096kB = 471600kB
#In this example I used SysRq again to show what each number in /proc/buddyinfo is referring

根据以上文章理解内存碎片是怎么回事了。
471600kb就是可用的内存（可以连续分配的内存地址空间），以4kb，8,16,32,64,128,256,512,1024,2048,4096kb为单位的所有可分配内存之和。

