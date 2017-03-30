---
title: '/proc/irq/{number}/smp_affinity'
date: 2017-03-30 13:56:55
tags: linux
---

在多 CPU 的环境中，还有一个中断平衡的问题，比如，网卡中断会教给哪个 CPU 处理，这个参数控制哪些 CPU 可以绑定 IRQ 中断。其中的 {number} 是对应设备的中断编号，可以用下面的命令找出：
cat /proc/interrupt
比如，一般 eth0 的 IRQ 编号是 16，所以控制 eth0 中断绑定的 /proc 文件名是 /proc/irq/16/smp_affinity。上面这个命令还可以看到某些中断对应的CPU处理的次数，缺省的时候肯定是不平衡的。
设置其值的方法很简单，smp_affinity 自身是一个位掩码（bitmask），特定的位对应特定的 CPU，这样，01 就意味着只有第一个 CPU 可以处理对应的中断，而 0f（0x1111）意味着四个 CPU 都会参与中断处理。
几乎所有外设都有这个参数设置，可以关注一下。
这个数值的推荐设置，其实在很大程度上，让专门的CPU处理专门的中断是效率最高的，比如，给磁盘IO一个CPU，给网卡一个CPU，这样是比较合理的。


[SMP IRQ Affinity](https://cs.uwaterloo.ca/~brecht/servers/apic/SMP-affinity.txt)

Background:  

Whenever a piece of hardware, such as disk controller or ethernet card, 
needs attention from the CPU, it throws an interrupt.  The interrupt tells
the CPU that something has happened and that the CPU should drop what
it's doing to handle the event.  In order to prevent mutliple devices from
sending the same interrupts, the IRQ system was established where each device
in a computer system is assigned its own special IRQ so that its interrupts
are unique.

Starting with the 2.4 kernel, Linux has gained the ability to assign certain
IRQs to specific processors (or groups of processors).  This is known
as SMP IRQ affinity, and it allows you control how your system will respond
to various hardware events.  It allows you to restrict or repartition
the work load that you server must do so that it can more efficiently do
it's job.

Obviously, in order for this to work, you will need a system that has more
than one processor (SMP).  You will also need to be running a 2.4 or higher
kernel.

Some brief and very bare information on SMP IRQ affinity is provided in
the kernel source tree of the 2.4 kernel in the file:

    /usr/src/linux-2.4/Documentation/IRQ-affinity.txt


How to use it:

SMP affinity is controlled by manipulating files in the /proc/irq/ directory.
In /proc/irq/ are directories that correspond to the IRQs present on your
system (not all IRQs may be available). In each of these directories is
the "smp_affinity" file, and this is where we will work our magic.

The first order of business is to figure out what IRQ a device is using.
This information is available in the /proc/interrupts file.  Here's a sample:

 [root@archimedes /proc]# cat /proc/interrupts 
            CPU0       CPU1       CPU2       CPU3       
   0:    4865302    5084964    4917705    5017077    IO-APIC-edge  timer
   1:        132        108        159        113    IO-APIC-edge  keyboard
   2:          0          0          0          0          XT-PIC  cascade
   8:          0          1          0          0    IO-APIC-edge  rtc
  10:          0          0          0          0   IO-APIC-level  usb-ohci
  14:          0          0          1          1    IO-APIC-edge  ide0
  24:      87298      86066      86012      86626   IO-APIC-level  aic7xxx
  31:      93707     106211     107988      93329   IO-APIC-level  eth0
 NMI:          0          0          0          0 
 LOC:   19883500   19883555   19883441   19883424 
 ERR:          0
 MIS:          0


As you can see, this is a 4 processor machine.  The first column (unlabelled)
lists the IRQs used on the system.  The rows with letters (ie, "NMI", "LOC")
are parts of other drivers used on the system and aren't really accessible
to us, so we'll just ignore them.

The second through fifth columns (labelled CPU0-CPU3) show the number of times
the corresponding process has handled an interrupt from that particular IRQ.
For example, all of the CPUs have handled roughly the same number of interrupts
for IRQ 24 (around 86,000 with CPU0 handling a little over 87,000).

The sixth column lists whether or not the device driver associated with the 
interrupt supports IO-APIC (see /usr/src/linux/Documentation/i386/IO-APIC.txt
for more information).  The only reason to look at this value is that 
SMP affinity will only work for IO-APIC enabled device drivers.  For
example, we will not be able to change the affinity for the "cascade"
driver (IRQ 2) because it doesn't support IO-APIC.

Finally, the seventh and last column lists the driver or device that is 
associated with the interrupt.  In the above example, our ethernet card
(eth0) is using IRQ 31, and our SCSI controller (aic7xxx) is using IRQ 24.

The first and last columns are really the only ones we're interested in here.
For the rest of this example, I'm going to assume that we want to adjust
the SMP affinity for th SCSI controller (IRQ 24).

Now that we've got the IRQ, we can change the processor affinity.  To
do this, we'll go into the /proc/irq/24/ directory, and see what the
affinity is currently set to:

 [root@archimedes Documentation]# cat /proc/irq/24/smp_affinity 
 ffffffff

This is a bitmask that represents which processors any interrupts on IRQ
24 should be routed to.  Each field in the bit mask corresponds to a processor.
The number held in the "smp_affinity" file is presented in hexadecimal format,
so in order to manipulate it properly we will need to convert our bit patterns
from binary to hex before setting them in the proc file.

Each of the "f"s above represents a group of 4 CPUs, with the rightmost
group being the least significant.  For the purposes of our discussion,
we're going to limit ourselves to only the first 4 CPUs (although we can
address up to 32).

In short, this means you only have to worry about the rightmost "f" and you
can assume everything else is a "0" (ie, our bitmask is "0000000f").

"f" is the hexadecimal represenatation for the decimal number 15 (fifteen)
and the binary pattern of "1111".  Each of the places in the binary pattern
corresponds to a CPU in the server, which means we can use the following
chart to represent the CPU bit patterns:

            Binary       Hex 
    CPU 0    0001         1 
    CPU 1    0010         2
    CPU 2    0100         4
    CPU 3    1000         8

By combining these bit patterns (basically, just adding the Hex values), we
can address more than one processor at a time.   For example, if I wanted
to talk to both CPU0 and CPU2 at the same time, the result is:

            Binary       Hex 
    CPU 0    0001         1 
  + CPU 2    0100         4
    -----------------------
    both     0101         5

If I want to address all four of the processors at once, then the result is:

            Binary       Hex 
    CPU 0    0001         1 
    CPU 1    0010         2
    CPU 2    0100         4
  + CPU 3    1000         8
    -----------------------
    both     1111         f

(Remember that we use the letters "a" through "f" to represent the numbers
 "10" to "15" in hex notation).

Given that, we now know that if we have a four processor system, we can 
assign any of 15 different CPU combinations to an IRQ (it would be 16, but
it isn't legal to assign an IRQ affinity of "0" to any IRQ... if you try,
Linux will just ignore your attempt).

So.  Now we get to the fun part.  Remember in our /proc/interrupts listing
above that all four of our CPUs had handled the close to the same amount of
interrupts for our SCSI card?  We now have the tools needed to limit managing
the SCSI card to just one processor and leave the other three free to
concentrate on doing other tasks.   Let's assume that we want to dedicate
our first CPU (CPU0) to handling the SCSI controller interrupts.  To do this,
we would simply run the following command:

 [root@archimedes /proc]# echo 1 > /proc/irq/24/smp_affinity 
 [root@archimedes /proc]# cat /proc/irq/24/smp_affinity 
 00000001

Now, let's test it out and see what happens:

 [root@archimedes /proc]# cd /tmp/
 [root@archimedes /tmp]# tar -zcf test.tgz /usr/src/linux-2.4.2 
 tar: Removing leading `/' from member names
 [root@archimedes /tmp]# tar -zxf test.tgz && rm -rf usr/
 [root@archimedes /tmp]# tar -zxf test.tgz && rm -rf usr/
 [root@archimedes /tmp]# tar -zxf test.tgz && rm -rf usr/
 [root@archimedes /tmp]# tar -zxf test.tgz && rm -rf usr/
 [root@archimedes /tmp]# tar -zxf test.tgz && rm -rf usr/
 [root@archimedes /tmp]# cat /proc/interrupts | grep 24:
  24:      99719      86067      86012      86627   IO-APIC-level  aic7xxx

Compare that to the previous run without having the IRQ bound to CPU0:

  24:      87298      86066      86012      86626   IO-APIC-level  aic7xxx

All of the interrupts from the disk controller are now handled exclusively
by the first CPU (CPU0), which means that our other 3 proccessors are free
to do other stuff now.

Finally, it should be pointed out that if you decide you no longer want
SMP affinity and would rather have the system revert back to the old set up,
then you can simply do:

 [root@archimedes /tmp]# cat /proc/irq/prof_cpu_mask >/proc/irq/24/smp_affinity

This will reset the "smp_affinity" file to use all "f"s, and will return to
the load sharing arrangement that we saw earlier.


What can I use it for?

- "balance" out multiple NICs in a multi-processor machine.  By tying a single
  NIC to a single CPU, you should be able to scale the amount of traffic
  your server can handle nicely.

- database servers (or servers with lots of disk storage) that also have
  heavy network loads can dedicate a CPU to their disk controller and assign
  another to deal with the NIC to help improve response times.


Can I do this with processes?

At this time, no.

