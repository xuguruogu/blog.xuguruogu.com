---
title: 网卡队列RSS/RPS/RFS
date: 2017-03-30 15:25:27
tags: [linux, tcp]
---

网卡优化
RSS receive side scaling，网卡多队列，需要硬件支持。网卡接收到网络数据包后，要发送一个硬件中断，通知CPU取数据包。默认配置，都是由CPU0去做。
RPS receive packet steering，向某个CPU发送一个软中断，来接收数据包，并递交给应用程序。
RFS receive flow streering，维护两种hash表，实现将软中断分散到多颗CPU上去处理。
 
- 选择支持msi-x中断方式的网卡类型
#lspci –v
- 网卡需要支持多队列
#lspci –vvv
如果有MSI-X && Enable+ && TabSize > 1，则该网卡是多队列网卡
- 2.6.35 以后的内核版本都支持google的RPS/RFS补丁，RHEL6.1以后。这个功能默认关闭需要手工开启
开启RPS(两颗4c的CPU)
#echo ff > /sys/class/net/eth0/queues/rx-0/rps_cpus
开启RFS(内存大的机器可以设置大于4096)
#echo 4096 > /sys/class/net/eth0/queues/rx-0/rps_flow_cnt
4096*N(N网卡队列数# cat /proc/interrupts | grep eth0)
#echo 32768 > /proc/sys/net/core/rps_sock_flow_entries
 
- <http://blog.netzhou.net/?p=181>
- <http://blog.csdn.net/turkeyzhou/article/details/7528182>
- <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/index.html>
