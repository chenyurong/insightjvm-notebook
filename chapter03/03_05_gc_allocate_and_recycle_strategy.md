# 内存分配策略

JVM在进行内存分配的时候回有一些固定的策略，它们分别是：

- 对象优先在Eden分配
- 大对象直接进入老年代
- 长期存活的进入老年代
- 动态对象年龄判断

**新生代GC（Minor GC）**

指发生在新生代的垃圾手机动作，因为新生代的对象大多都是朝生夕灭，所以Minor GC相对比较频繁，一般回收速度也比较快。

**老年代GC（Major GC/Full GC）**

指发生在老年代的GC，出现了Major GC，经常会伴随着至少一次Minor GC（但非绝对）。Major GC的速度一般会比Minor GC慢10倍以上。


## 对象优先在Eden分配

大多数情况下，对象优先在新生代的Eden区中分配。当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC。在进行Minor GC时，如果Survivor分区的内存不够，那么GC会将对象直接移动到老年代。下面我们通过一个例子的GC日志来验证这个内存分配策略。

```java
private static final int _1MB = 1024 * 1024;
/**
 * 1.对象优先在Eden分配
 * VM参数：-XX:+UseSerialGC -verbose:gc  -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8
 * -XX:+UseSerialGC  强制使用Serial+SerialOld收集器组合
 * -verbose:gc       输出虚拟机中GC的详细情况
 * -Xms20M -Xmx20M -Xmn10M  堆初始化大小20M，堆最大20M，新生代大小10M
 * -XX:+PrintGCDetails   垃圾回收时打印内存回收日志
 * -XX:SurvivorRatio=8   Eden:Survivor=8:1
 */
public static void testAllocation() {
    byte[] allocation1, allocation2, allocation3, allocation4;
    allocation1 = new byte[2 * _1MB];
    allocation2 = new byte[2 * _1MB];
    allocation3 = new byte[2 * _1MB];
    allocation4 = new byte[4 * _1MB];   //出现一次Minor GC
}
```

GC日志：

```bash
[GC[DefNew: 7471K->382K(9216K), 0.0082590 secs] 7471K->6526K(19456K), 0.0083080 secs] [Times: user=0.00 sys=0.01, real=0.01 secs] 
Heap
 def new generation   total 9216K, used 4727K [0x00000007f9a00000, 0x00000007fa400000, 0x00000007fa400000)
  eden space 8192K,  53% used [0x00000007f9a00000, 0x00000007f9e3e698, 0x00000007fa200000)
  from space 1024K,  37% used [0x00000007fa300000, 0x00000007fa35f888, 0x00000007fa400000)
  to   space 1024K,   0% used [0x00000007fa200000, 0x00000007fa200000, 0x00000007fa300000)
 tenured generation   total 10240K, used 6144K [0x00000007fa400000, 0x00000007fae00000, 0x00000007fae00000)
   the space 10240K,  60% used [0x00000007fa400000, 0x00000007faa00030, 0x00000007faa00200, 0x00000007fae00000)
 compacting perm gen  total 21248K, used 2944K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)
   the space 21248K,  13% used [0x00000007fae00000, 0x00000007fb0e0170, 0x00000007fb0e0200, 0x00000007fc2c0000)
No shared spaces configured.
```

我们为JVM分配了20M的堆大小，其中新生代10M，老年代10M。其中老年代10M中，Eden:Survivor=8:1，即Eden为8M，两个Survivor各为1M。

当程序为allocation4分配4M内存的时候，JVM已经在Eden中分配了6M的内存，JVM分析发现Eden区不足以再分配4M内存，于是发起了一次Minor GC。JVM进行Minor GC时发现没有可回收的内存，并且Survivor的1M内存不足以存放原先的6M存活对象，于是把这6M对象直接挪到了老年代。可以看到GC日志中的“tenured generation   total 10240K, used 6144K”表面老年代10M内存用了6M左右。之后JVM再在Eden区为allocation4指向的对象分配4M内存，对应GC日志中的“eden space 8192K,  53% used”。

## 大对象直接进入老年代

在程序中有时候会出现一些大对象，它们的存活时间也非常短，这时候如果将它们放在Eden区，那么会经常导致Eden区内存不足进而导致Minor GC。对于这种情况，JVM提供了`-XX:PretenureSizeThreshold`参数来设置超过一定大小的对象直接在老年代分配内存。

```
private static final int _1MB = 1024 * 1024;
/**
 * 2.大对象直接进入老年代
 * VM参数：-XX:+UseSerialGC -verbose:gc  -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:PretenureSizeThreshold=3145728
 * -XX:PretenureSizeThreshold=3145728(这里不能写成MB的形式）  超过3MB时，直接在老年代分配
 */
public static void testPretenureSizeThreshold() {
    byte[] allocation;
    allocation = new byte[4 * _1MB];    //直接分配在老年代中
}
```

程序中设置了JVM参数：`-XX:PretenureSizeThreshold=3145728`，如果对象大小超过3M，那么将直接在老年代进行内存分配。下面是程序的GC日志：

```
Heap
 def new generation   total 9216K, used 1491K [0x00000007f9a00000, 0x00000007fa400000, 0x00000007fa400000)
  eden space 8192K,  18% used [0x00000007f9a00000, 0x00000007f9b74e88, 0x00000007fa200000)
  from space 1024K,   0% used [0x00000007fa200000, 0x00000007fa200000, 0x00000007fa300000)
  to   space 1024K,   0% used [0x00000007fa300000, 0x00000007fa300000, 0x00000007fa400000)
 tenured generation   total 10240K, used 4096K [0x00000007fa400000, 0x00000007fae00000, 0x00000007fae00000)
   the space 10240K,  40% used [0x00000007fa400000, 0x00000007fa800010, 0x00000007fa800200, 0x00000007fae00000)
 compacting perm gen  total 21248K, used 2899K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)
   the space 21248K,  13% used [0x00000007fae00000, 0x00000007fb0d4c40, 0x00000007fb0d4e00, 0x00000007fc2c0000)
No shared spaces configured.
```

从日志中可以发现新生代`def new generation   total 9216K, used 1491K`只有少量的内存分配，而老年代内存`tenured generation   total 10240K, used 4096K`则有4M的内存占用。

## 长期存活的进入老年代

如果一个对象在新生代经历过的垃圾收集次数达到了一定的数值，那么该对象就会被移动到老年代。JVM中通过`-XX:MaxTenuringThreshold`参数可以设置新生代年龄上限。

```
/**
 * 长期存活的对象进入老年代
 * VM参数：-XX:+UseSerialGC -verbose:gc  -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=1 -XX:+PrintTenuringDistribution
 * -XX:MaxTenuringThreshold=1 设置新生代年龄上限
 * -XX:PrintTenuringDistribution 打印长期存活对象的年龄等信息
 */
public static void testTenuringThreshold() {
    byte[] allocation1, allocation2, allocation3;
    allocation1 = new byte[_1MB];
    //什么时候进入老年代取决于XX:MaxTenuringThreshold设置
    allocation2 = new byte[4 * _1MB];
    allocation3 = new byte[4 * _1MB];
    allocation3 = null;
    allocation3 = new byte[4 * _1MB];
}
```

程序GC日志：

```
[GC[DefNew
Desired survivor size 524288 bytes, new threshold 1 (max 1)
- age   1:     386536 bytes,     386536 total
: 6447K->377K(9216K), 0.0052170 secs] 6447K->5497K(19456K), 0.0052430 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC[DefNew
Desired survivor size 524288 bytes, new threshold 1 (max 1)
- age   1:       4760 bytes,       4760 total
: 4641K->4K(9216K), 0.0014780 secs] 9761K->5481K(19456K), 0.0014950 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 4238K [0x00000007f9a00000, 0x00000007fa400000, 0x00000007fa400000)
  eden space 8192K,  51% used [0x00000007f9a00000, 0x00000007f9e22720, 0x00000007fa200000)
  from space 1024K,   0% used [0x00000007fa200000, 0x00000007fa201298, 0x00000007fa300000)
  to   space 1024K,   0% used [0x00000007fa300000, 0x00000007fa300000, 0x00000007fa400000)
 tenured generation   total 10240K, used 5477K [0x00000007fa400000, 0x00000007fae00000, 0x00000007fae00000)
   the space 10240K,  53% used [0x00000007fa400000, 0x00000007fa959560, 0x00000007fa959600, 0x00000007fae00000)
 compacting perm gen  total 21248K, used 2932K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)
   the space 21248K,  13% used [0x00000007fae00000, 0x00000007fb0dd338, 0x00000007fb0dd400, 0x00000007fc2c0000)
No shared spaces configured.
```

##动态对象年龄判断

为了能更好地适应不同程序的内存状况，虚拟机并不是永远地要求对象的年龄必须达到了MaxTenuringThreshold才能晋升老年代。如果在Survivor空间中相同年龄所有对象的大小综合大于Survivor空间的一般，年龄大于或等于该年龄的对象就可以直接进入老年代，无序等到MaxTenuringThreshold中要求的年龄。
