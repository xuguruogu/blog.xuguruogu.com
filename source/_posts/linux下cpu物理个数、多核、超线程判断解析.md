---
title: linux下cpu物理个数、多核、超线程判断解析
date: 2017-03-30 11:10:29
tags: linux
---
在Linux体系中，供给了proc文件体系显示体系的软硬件信息。若是想懂得体系中CPU的供给商和相干设备信息，则可以经由过程/proc/cpuinfo文件获得。本文章针对该文件进行简单的总结。

基于指令集（ISA）的CPU产生的/proc/cpuinfo文件不一样，基于X86指令集CPU的/proc/cpuinfo文件包含如下内容：

```
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 23
model name      : Intel(R) Xeon(R) CPU           E5430  @ 2.66GHz
stepping        : 10
cpu MHz         : 2666.890
cache size      : 6144 KB
physical id     : 0
siblings        : 4
core id         : 0
cpu cores       : 4
apicid          : 0
initial apicid  : 0
fpu             : yes
fpu_exception   : yes
cpuid level     : 13
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx lm constant_tsc arch_perfmon pebs bts rep_good aperfmperf pni tm2 ssse3 lahf_lm dts
bogomips        : 5333.78
clflush size    : 64
cache_alignment : 64
address sizes   : 38 bits physical, 48 bits virtual
power management:
```

以上输出项的含义如下：

```
processor　：体系中逻辑处理惩罚核的编号。对于单核处理惩罚器，则课认为是其CPU编号，对于多核处理惩罚器则可所以物理核、或者应用超线程技巧虚拟的逻辑核
vendor_id　：CPU建造商      
cpu family　：CPU产品系列代号
model　　　：CPU属于其系列中的哪一代的代号
model name：CPU属于的名字及其编号、标称主频
stepping　  ：CPU属于建造更新版本
cpu MHz　  ：CPU的实际应用主频
cache size   ：CPU二级缓存大小
physical id   ：单个CPU的标号
siblings       ：单个CPU逻辑物理核数
core id        ：当前物理核在其所处CPU中的编号，这个编号不必然连气儿
cpu cores    ：该逻辑核所处CPU的物理核数
apicid          ：用来区分不合逻辑核的编号，体系中每个逻辑核的此编号必定不合，此编号不必然连气儿
fpu             ：是否具有浮点运算单位（Floating Point Unit）
fpu_exception  ：是否支撑浮点策画异常
cpuid level   ：履行cpuid指令前，eax存放器中的值，按照不合的值cpuid指令会返回不合的内容
wp             ：注解当前CPU是否在内核态支撑对用户空间的写保护（Write Protection）
flags          ：当前CPU支撑的功能
bogomips   ：在体系内核启动时粗略测算的CPU速度（Million Instructions Per Second）
clflush size  ：每次刷新缓存的大小单位
cache_alignment ：缓存地址对齐单位
address sizes     ：可接见地址空间位数
power management ：对能源经管的支撑
```

CPU信息中flags各项含义：

```
fpu： Onboard （x87） Floating Point Unit
vme： Virtual Mode Extension
de： Debugging Extensions
pse： Page Size Extensions
tsc： Time Stamp Counter: support for RDTSC and WRTSC instructions
msr： Model-Specific Registers
pae： Physical Address Extensions: ability to access 64GB of memory; only 4GB can be accessed at a time though
mce： Machine Check Architecture
cx8： CMPXCHG8 instruction
apic： Onboard Advanced Programmable Interrupt Controller
sep： Sysenter/Sy***it Instructions; SYSENTER is used for jumps to kernel memory during system calls， and SY***IT is used for jumps： back to the user code
mtrr： Memory Type Range Registers
pge： Page Global Enable
mca： Machine Check Architecture
cmov： CMOV instruction
pat： Page Attribute Table
pse36： 36-bit Page Size Extensions: allows to map 4 MB pages into the first 64GB RAM， used with PSE.
pn： Processor Serial-Number; only available on Pentium 3
clflush： CLFLUSH instruction
dtes： Debug Trace Store
acpi： ACPI via MSR
mmx： MultiMedia Extension
fxsr： FXSAVE and FXSTOR instructions
sse： Streaming SIMD Extensions. Single instruction multiple data. Lets you do a bunch of the same operation on different pieces of input： in a single clock tick.
sse2： Streaming SIMD Extensions-2. More of the same.
selfsnoop： CPU self snoop
acc： Automatic Clock Control
IA64： IA-64 processor Itanium.
ht： HyperThreading. Introduces an imaginary second processor that doesn’t do much but lets you run threads in the same process a  bit quicker.
nx： No ute bit. Prevents arbitrary code running via buffer overflows.
pni： Prescott New Instructions aka. SSE3
vmx： Intel Vanderpool hardware virtualization technology
svm： AMD “Pacifica” hardware virtualization technology
lm： “Long Mode，” which means the chip supports the AMD64 instruction set
tm： “Thermal Monitor” Thermal throttling with IDLE instructions. Usually hardware controlled in response to CPU temperature.
tm2： “Thermal Monitor 2″ Decrease speed by reducing multipler and vcore.
est： “Enhanced SpeedStep”
```

- 查看CPU信息命令
cat /proc/cpuinfo
- 查看内存信息命令
cat /proc/meminfo
- 查看硬盘信息命令
fdisk -l

- 查询体系CPU的物理个数
    cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc -l
- 查询体系具有几许个逻辑核
    cat /proc/cpuinfo | grep "processor" | wc -l
- 查询体系CPU的物理核数
    cat /proc/cpuinfo | grep "cpu cores" | uniq
- 查询体系CPU是否启用超线程
    cat /proc/cpuinfo | grep -e "cpu cores"  -e "siblings" | sort | uniq
- 查询CPU的主频
    cat /proc/cpuinfo |grep MHz|uniq
    输出举例：
    cpu cores    : 6
    siblings    　: 6
- 查看当前系统内核信息
    uname -a
    Linux localhost.localdomain 2.6.32-220.el6.x86_64 #1 SMP Tue Dec 6 19:48:22 GMT2011x86_64 x86_64 x86_64 GNU/Linux
- 查看当前操作系统发行版信息：
    cat /etc/issue | grep Linux
    Red Hat Enterprise Linux AS release 4 (Nahant Update 5)

- 查看逻辑CPU、CPU型号
    cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
    8  Intel(R) Xeon(R) CPU            E5410   @ 2.33GHz

- 查看物理核心
    # cat /proc/cpuinfo | grep physical | uniq -c
    4 physical id      : 0
    4 physical id      : 1
    (说明实际上是两颗4核的CPU)

- 32/64位
    # getconf LONG_BIT
    32
    (说明当前CPU运行在32bit模式下, 但不代表CPU不支持64bit)

- 
    # cat /proc/cpuinfo | grep flags | grep ' lm ' | wc -l
    8
    (结果大于0, 说明支持64bit计算. lm指long mode, 支持lm则是64bit)

- 若是cpu cores数量和siblings数量一致，则没有启用超线程，不然超线程被启用。

查询体系CPU是否支撑某项功能，则根以上类似，输出成果进行sort， uniq和grep就可以获得成果。
processor 条目包括这一逻辑处理器的唯一标识符。 
physical id 条目包括每个物理封装的唯一标识符。 
core id 条目保存每个内核的唯一标识符。 
siblings 条目列出了位于相同物理封装中的逻辑处理器的数量。 
cpu cores 条目包含位于相同物理封装中的内核数量。 
如果处理器为英特尔处理器，则 vendor id 条目中的字符串是 GenuineIntel。
拥有相同 physical id 的所有逻辑处理器共享同一个物理插座。每个 physical id 代表一个唯一的物理封装。Siblings 表示位于这一物理封装上的逻辑处理器的数量。它们可能支持也可能不支持超线程（HT）技术。每个 core id 均代表一个唯一的处理器内核。所有带有相同 core id 的逻辑处理器均位于同一个处理器内核上。如果有一个以上逻辑处理器拥有相同的 core id 和 physical id，则说明系统支持超线程（HT）技术。如果有两个或两个以上的逻辑处理器拥有相同的 physical id，但是 core id 不同，则说明这是一个多内核处理器。cpu cores 条目也可以表示是否支持多内核。
例如，如果系统包含两个物理封装，每个封装中又包含两个支持超线程（HT）技术的处理器内核，则 /proc/cpuinfo 文件将包含此数据。（注：数据并不在表格中。）


?processor ?0 ?1 ?2 ?3 ?4 ?5 ?6 ?7
?physical id ?0 ?1 ?0 ?1 ?0 ?1 ?0 ?1
?core id ?0 2 1 ?3 ?0 ?2 ?1 ?3
?siblings ?4 ?4 ?4 ?4 ?4 ?4 ?4 ?4
?cpu cores ?2 ?2 ?2 ?2 ?2 ?2 ?2 ?2

此例说明逻辑处理器 0 和 4 驻留在物理封装 0 的内核 0 上。这就表示逻辑处理器 0 和 4 支持超线程（HT）技术。相同的工作可用于封装 0 内核 1 上的逻辑处理器 2 和 6，封装 1 内核 2 上的逻辑处理器 1 和 5，以及封装 1 内核 3 上的逻辑处理器 3 和 7。此系统支持超线程（HT）技术，因为两个逻辑处理器共享同一个内核。有两种方式可以确定是否支持多内核。由于内核 0 和 1 存在于封装 0 上，而内核 2 和 3 存在于封装 1 上，所以这是一个多内核系统。此外，cpu cores 条目为 2，也说明有两个内核驻留在物理封装中。这是一个多路系统，因为有两个封装。值得注意的是 physical id 和 core id 的编号可能是也可能不是连续的。系统上有两个物理封装并不罕见，而且 physical id 等于 0 和 3

CPU ID
CPU ID是CPU生产厂家为识别不同类型的CPU，而为CPU制订的不同的单一的代码；不同厂家的CPU，其CPU ID定义也是不同的；如 “0F24”（Inter处理器）、“681H”（AMD处理器），根据这些数字代码即可判断CPU属于哪种类型，这就是一般意义上的CPU ID。 由于计算机使用的是十六进制，因此CPU ID也是以十六进制表示的。Inter处理器的CPU ID一共包含四个数字，如“0F24”，从左至右分别表示 Type（类型）、Family（系列）、Mode（型号）和Stepping（步进编号）。从CPUID为“068X”的处理器开始，Inter另外增 加了Brand ID（品种标识）用来辅助应用程序识别CPU的类型，因此根据“068X”CPUID还不能正确判别Pentium和Celerom处理 器。必须配合Brand ID来进行细分。AMD处理器一般分为三位，如“681”，从左至右分别表示为Family（系列）、Mode（型号）和 Stepping（步进编号）。

Type（类型） 
类型标识用来区别INTEL微处理器是用于由最终用户安装，还是由专业个人计算机系 统集成商、服务公司或制作商安装；数字“1”标识所测试的微处理器是用于由用户安装的；数字“0”标识所测试的微处理器是用于由专业个人计算机系统集成 商、服务公司或制作商安装的。我们通常使用的INTEL处理器类型标识都是“0”，“0F24”CPUID就属于这种类型。

Family（系列） 
系 列标识可用来确定处理器属于那一代产品。如6系列的INTEL处理器包括Pentium Pro、Pentium II、 Pentium II Xeon、Pentium III和Pentium III Xeon处理器。5系列（第五代）包括Pentium处理器和采用 MMX技术的Pentium处理器。AMD的6系列实际指有K7系列CPU，有DURON和ATHION两大类。最新一代的 INTEL Pentium 4系列处理器（包括相同核心的Celerom处理器）的系列值为“F”

Mode（型号） 
型号标识可用来 确定处理器的制作技术以及属于该系列的第几代设计（或核心），型号与系列通常是相互配合使用的，用于确定计算机所安装的处理器是属于某系列处理器的哪种特 定类型。如可确定Celerom处理器是Coppermine还是Tualutin核心；Athlon XP处理器是Paiomino还是 Thorouhgbred核心。

Stepping（步进编号） 
步进编号用来标识处理器的设计或制作版本，有助于控制和跟踪处理器的更 改，步进还可以让最终用户更具体地识别其系统安装的处理器版本，确定微处理器的内部设计或制作特性。步进编号就好比处理器的小版本号，如CPUID为 “686”和“686A”就好比WINZIP8.0和8.1的关系。步进编号和核心步进是密切联系的。如CPUID为“686”的Pentium III 处理器是cCO核心，而“686A”表示的是更新版本cD0核心。

Brand ID（品种标识） 
INTEL从Coppermine核心的处理器开始引入Brand ID作为CPU的辅助识别手段。如我们通过Brand ID可以识别出处理器究竟是Celerom还是Pentium 4。
 
在LINUX系统中，一颗超线程CPU，将被识别为两颗CPU，一颗双核CPU，也被识别为两颗CPU，而一颗双核超线程CPU，会被认为是4颗CPU。那么，我们如何确定我们机器的CPU数量呢？

仔细查看/proc/cpuinfo我们会发现以下信息：
- physical id代表每颗物理CPU的ID，有几个CPU ID，就有几颗物理CPU。
- siblings区别出了超线程CPU中的逻辑CPU核心，一颗超线程CPU，其physical id是一样的，但是siblings是不同的。
- core id和cpu cores用来对双核（多核心）CPU进行区分的，CPU cores表示这颗CPU有几个核心，而core id用来表示CPU的各个核心的。

例如：如何区分一颗双核超线程CPU？
cat /etc/proc/cpuinfo
{
physical id=0  （1颗物理CPU）
  [
  core id=0    （双核CPU中的第一个核心）
  cpu cores=2  （双核CPU）
    siblings=0 （此核心中的第一个逻辑CPU）
    siblings=1 （此核心中的另一个逻辑CPU）
  ]
  [
  core id=1    （双核CPU中的另一个核心）
  cpu cores=2  （双核CPU）
    siblings=0 （此核心中的第一个逻辑CPU）
    siblings=1 （此核心中的另一个逻辑CPU）
  ]
}
