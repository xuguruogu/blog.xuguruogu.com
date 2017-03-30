layout: linux性能测试
title: mpstat命令
date: 2017-03-30 15:35:35
tags: [linux, cpu]
---

mpstat是MultiProcessor Statistics的缩写，是实时系统监控工具。其报告与CPU的一些统计信息，这些信息存放在/proc/stat文件中。在多CPUs系统里，其不但能查看所有CPU的平均状况信息，而且能够查看特定CPU的信息。
下面只介绍mpstat与CPU相关的参数，mpstat的语法如下：

    Usage: mpstat [ options... ] [ <interval> [ <count> ] ]
    Options are:
    [ -P { <cpu> | ALL } ] [ -V ]

参数的含义如下：
参数 解释
-P {|ALL} 表示监控哪个CPU， cpu在[0,cpu个数-1]中取值
internal 相邻的两次采样的间隔时间
count 采样的次数，count只能和delay一起使用

当没有参数时，mpstat则显示系统启动以后所有信息的平均值。有interval时，第一行的信息自系统启动以来的平均信息。
从第二行开始，输出为前一个interval时间段的平均信息。

与CPU有关的输出的含义如下：
参数 解释 从/proc/stat获得数据
CPU 处理器ID
user 在internal时间段里，用户态的CPU时间（%），不包含 nice值为负 进程 (usr/total)*100  
nice 在internal时间段里，nice值为负进程的CPU时间（%）   (nice/total)*100  
system 在internal时间段里，核心时间（%）   (system/total)*100
iowait 在internal时间段里，硬盘IO等待时间（%） (iowait/total)*100
irq 在internal时间段里，硬中断时间（%）      (irq/total)*100
soft 在internal时间段里，软中断时间（%）    (softirq/total)*100
idle 在internal时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间（%）(idle/total)*100
intr/s 在internal时间段里，每秒CPU接收的中断的次数intr/total)*100
CPU总的工作时间=total_cur=user+system+nice+idle+iowait+irq+softirq
total_pre=pre_user+ pre_system+ pre_nice+ pre_idle+ pre_iowait+ pre_irq+ pre_softirq
user=user_cur – user_pre
total=total_cur-total_pre
其中_cur 表示当前值，_pre表示interval时间前的值。上表中的所有值可取到两位小数点。

范例1：average mode (粗略信息)
当mpstat不带参数时，输出为从系统启动以来的平均值。

    [root@C44 ~]#  mpstat
    Linux 2.6.14.7-selinux1-WR1.4aq_cgl (MSP)       07/26/12

    12:47:05     CPU   %user   %nice    %sys %iowait    %irq   %soft   %idle    intr/s
    12:47:05     all    2.98    0.00    2.68    2.12    0.05    0.31   91.87    391.82

范例2: 每2秒产生了2个处理器的统计数据报告
下面的命令可以每2秒产生了2个处理器的统计数据报告，一共产生三个interval 的信息，然后再给出这三个interval的平
均信息。默认时，输出是按照CPU 号排序。第一个行给出了从系统引导以来的所有活跃数据。接下来每行对应一个处理器的
活跃状态。。


    [root@C44 ~]#  mpstat -P ALL 2 3
    Linux 2.6.14.7-selinux1-WR1.4aq_cgl (MSP)       07/26/12

    12:47:11     CPU   %user   %nice    %sys %iowait    %irq   %soft   %idle    intr/s
    12:47:13     all    1.51    0.00    0.76    0.00    0.00    0.25   97.48    296.50
    12:47:13       0    2.50    0.00    2.00    0.00    0.00    0.50   95.00    296.50
    12:47:13       1    1.00    0.00    0.00    0.00    0.00    0.50   98.00      0.00

    12:47:13     CPU   %user   %nice    %sys %iowait    %irq   %soft   %idle    intr/s
    12:47:15     all    0.50    0.00    0.25    0.00    0.00    0.00   99.24    295.45
    12:47:15       0    1.01    0.00    0.00    0.00    0.00    0.00   98.99    295.45
    12:47:15       1    0.00    0.00    0.00    0.00    0.00    0.00  100.00      0.00

    12:47:15     CPU   %user   %nice    %sys %iowait    %irq   %soft   %idle    intr/s
    12:47:17     all    0.51    0.00    0.76    0.25    0.00    0.25   98.23    299.49
    12:47:17       0    1.01    0.00    1.01    0.00    0.00    0.51   97.47    299.49
    12:47:17       1    0.00    0.00    1.01    0.00    0.00    0.00   99.49      0.00

    Average:     CPU   %user   %nice    %sys %iowait    %irq   %soft   %idle    intr/s
    Average:     all    0.84    0.00    0.59    0.08    0.00    0.17   98.32    297.15
    Average:       0    1.51    0.00    1.01    0.00    0.00    0.34   97.15    297.15
    Average:       1    0.34    0.00    0.34    0.00    0.00    0.17   99.16      0.00
