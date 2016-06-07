<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [垃圾回收器](#%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8)
  - [新生代垃圾回收器](#%E6%96%B0%E7%94%9F%E4%BB%A3%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8)
    - [Serial收集器](#serial%E6%94%B6%E9%9B%86%E5%99%A8)
    - [ParNew收集器](#parnew%E6%94%B6%E9%9B%86%E5%99%A8)
    - [Parallel Scavenge收集器](#parallel-scavenge%E6%94%B6%E9%9B%86%E5%99%A8)
  - [老年代垃圾回收器](#%E8%80%81%E5%B9%B4%E4%BB%A3%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8)
    - [Serial Old收集器](#serial-old%E6%94%B6%E9%9B%86%E5%99%A8)
    - [Parallel Old收集器](#parallel-old%E6%94%B6%E9%9B%86%E5%99%A8)
    - [CMS](#cms)
  - [G1收集器](#g1%E6%94%B6%E9%9B%86%E5%99%A8)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 垃圾回收器

垃圾回收器根据不同内存区域，有不同的垃圾回收器。新生代回收器主要有：Serial收集器、ParNew收集器、Parallel Scanvenge收集器。老年代收集器有：Serial Old收集器、Parallel Old收集器、CMS收集器。而G1收集器则是集新生代和老年代内存回收功能于一身的收集器。

## 新生代垃圾回收器

### Serial收集器

Serial收集器是最基本、发展历史最悠久的收集器，在JDK1.3.1之前是虚拟机新生代收集的唯一选择。

Serial收集器是一个单线程收集器，但其单线程不仅仅体现在其只启动一条线程进行垃圾回收，也体现在当其进行垃圾回收的时候，其他所有线程都必须停止执行（Stop The World, STW）。这种方式其实就像你妈妈在给你打扫房间，那么当她打扫的时候，你就必须停止仍垃圾，不然这怎么打扫得完。

Serial虽然只能以单线程的方式运行，但正是因为其单线程运行减少了线程之间的开销，所以对于限定单个CPU的环境来说，Serial收集器可以获得最高的单线程收集效率。所以对于运行在Client模式下的虚拟机来收，Serial收集器是一个很好的选择。

### ParNew收集器

ParNew收集器是多线程版本的Serial收集器，其可以启动多个线程进行垃圾回收，除此之外，其他方面都与Serial收集器完全相同，甚至它们的源码都是大部分相同的。与Serial收集器一样，ParNew收集器在进行垃圾回收的时候也必须让其他线程停止执行，即发生STW操作。

在单CPU环境下，ParNew收集器的效率比Serial收集器低。但随着CPU核数的不断提升，Parnew收集器的效率会不断提升，其优势会慢慢突显。因此它是许多运行在Server模式下的虚拟机中首选的新生代收集器。许多Server模式下虚拟机之所以选中ParNew收集器作为其新生代的收集器，除了其实多线程之外，另一个与性能无关的重要原因是，除了Serial收集器外，目前只有它能与CMS收集器配合工作。

### Parallel Scavenge收集器

Parallel Scavenge收集器可以说是ParNew的增强版，其同样使用复制算法，也是使用多线程进行垃圾回收。但其与ParNew的特别之处在于，Parallel Scanvenge收集器关注点与其他收集器不同，Parallel Scanvenge收集器的目标是达到一个**可控制的吞吐量**，所以Parallel Scanvenge也常被称为“吞吐量优先”收集器。

Parallel Scanvenge收集器与ParNew收集器的一个重要区别是Parallel Scanvenge具有GC自适应调节策略。虚拟机会根据当前系统的运行状况收集性能控制信息，动态调整这些参数以提供最合适的停顿时间或最大的吞吐量。

正因为Parallel Scanvenge收集器专注于吞吐量的优化，所以其适合用于后台运算而不需要太多交互的任务。

在虚拟机的具体实现中，新生代垃圾回收器还需要与老年代垃圾回收器搭配使用。但因为各种各样的原因。CMS（一种老年代垃圾回收器）只能与Serial收集器和ParNew收集器配合使用，而无法与Parallel Scanvenge收集器配合使用。

## 老年代垃圾回收器

### Serial Old收集器

Serial Old收集器是Serial收集器的老年代版本，它同样是一个单线程收集器，使用“标记-整理”算法。这个收集器的主要意义是提供给Client模式下的虚拟机使用。

### Parallel Old收集器

Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。这个收集器是在JDK1.6中才提供的，在这之前，如果虚拟机选择Parallel Scanvenge作为新生代收集器，那么因为Parallel Scanvenge不能和CMS选择器配合使用，所以其只能选择Serial Old收集器作为其老年代收集器。但因为老年代Serial Old收集器在服务端应用性能上（单线程）的拖累，所以导致其无法发挥出更强悍的性能。这导致了在老年代很大而且硬件比较高级的环境中，这种组合的吞吐量还不如ParNew加CMS的组合“给力”。

直到Parallel Old收集器出现后，“吞吐量优先”组合才有了较为合适的搭配。在注重吞吐量以及CPU资源敏感的组合，都可以优先考虑Parallel Scanvenge加Parallel Old收集器。

### CMS

CMS收集器使用多线程和“标记-清除”算法的一款收集器，其以获取最短回收停顿时间作为目标。因此CMS收集器非常符合B/S系统的服务端，因为这些系统往往重视服务的响应速度，希望系统停顿时间最短，给用户带来良好的使用体验。

CMS收集器的运作过程相对前面几种收集器来说更复杂一些，整个过程分为4个步骤，包括：

- 初始标记
- 并发标记
- 重新标记
- 并发清除

其中初始标记和并发标记两个步骤仍然需要“Stop The World”，初始标记只是简单的标记一下GC Roots能关联到的对象，速度很快，并发标记阶段就是进行GC Tracing的过程。而重新标记是为了修正并发标记期间因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录。

## G1收集器

G1收集器一款面向服务端应用的垃圾回收器，其使用时用于替换JDK1.5中发布的CMS收集器。与其他GC收集器相比，G1收集器有以下特点：

- **并发与并行**。G1能充分利用CPU、多喝环境下的硬件优势，使用多个CPU来缩短Stop The World的时间，减少程序的停顿。
- **分代收集**。与其他收集器一样，分代收集概念在G1收集器中得以保留。但G1收集器不需要其他收集器的配合就可以独立管理整个GC堆。
- **空间整理**。与CMS基于“标记-清除”算法不同，G1收集器整体上看来是机遇“标记-整理”算法实现的，而从局部上看来是基于“复制”算法实现的。但无论如何，这两种算法都意味着G1在运行期间不会产生内存碎片，适合程序长时间运行，不会因为遇到大对象而无法找到连续的内存进而促发GC。
- **可预测的停顿**。降低停顿时间是CMS和G1的共同关注点，但G1在CMS的基础上建立了可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不超过N毫秒。

G1收集器的回收过程其实与CMS的过程差不多，大致可以分为以下几个步骤：

- 初始标记
- 并发标记
- 最终标记
- 筛选回收

