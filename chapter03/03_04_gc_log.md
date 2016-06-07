<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [理解GC日志](#%E7%90%86%E8%A7%A3gc%E6%97%A5%E5%BF%97)
  - [JVM参数总结](#jvm%E5%8F%82%E6%95%B0%E6%80%BB%E7%BB%93)
  - [垃圾收集器参数总结](#%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8%E5%8F%82%E6%95%B0%E6%80%BB%E7%BB%93)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 理解GC日志

下面是两段典型的GC日志：

```
33.125: [GC [DefNew: 3324K->153K(3712K), 0.0025925 secs] 3324K->152K(11904K), 0.0031680 secs]   
100.667: [Full GC [Tenured: 0K->210K(10240K), 0.0149142 secs] 4603K->210K(19456K), [Perm:2999K->2999K(21248K)], 0.0150007 secs] [Times: user=0.01 sys=0.00, real=0.02 secs]
```

- 最前面的数字“33.124:” 和 “100.667:”- 代表了GC发生的时间，这个数字的含义是从Java虚拟机启动以来经过的秒数。
- GC日志开头的“[GC”和“[Full GC”说明了这次垃圾收集的停顿类型。如果有“Full”，说明这次GC是发生了Stop-The-World的。如果是调用System.gc()方法所触发的收集，那么在这里将显示“[Full GC(System)”
- DefNew、Tenured、Perm 表示GC发生的区域，不同GC收集器显示的名字是不同的。Serial收集器：DefNew；ParNew收集器：ParNew；Parallel Scavenge收集器：PSYoungGen
- 3324K->152K(3712K) 含义是GC前盖内存区域已使用的容量->GC后该内存区域已使用的容量（该内存区域总容量）
- 3324->152K(11904K) 韩式是GC前Java堆已使用容量->GC后Java堆已使用容量（Java堆总容量）

下面我们通过几个例子从浅到深讲解一下如何读懂GC日志。

这里我们所有的例子都将以下面的JVM参数运行：

```
VM参数：-XX:+UseSerialGC -verbose:gc  -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8
```

- -XX:+UseSerialGC  强制使用Serial+SerialOld收集器组合
- -verbose:gc       输出虚拟机中GC的详细情况
- -Xms20M -Xmx20M -Xmn10M  堆初始化大小20M，堆最大20M，新生代大小10M
- -XX:+PrintGCDetails   垃圾回收时打印内存回收日志
- -XX:SurvivorRatio=8   Eden:Survivor=8:1

所以最终JVM内存分配情况是：Eden区 8M，FromSurvivor 1M，ToSurvivor 1M，老年代 10M。

```java
private static final int _1MB = 1024 * 1024;
public static void simpleGC1(){
    byte[] allocation1, allocation2;
    allocation1 = new byte[4 * _1MB];
    allocation2 = new byte[4 * _1MB];  
}
```

当运行到`allocation2 = new byte[4 * _1MB];`时，此时Eden区已分配了4M+内存（因为还有其他对象会占用少部分空间），JVM检测到无法再在Eden空间给allocation2分配4M内存，于是发生一次Minor GC。在将allocation1放到Survivor中的时候发现Survivor无法容纳下4M的空间，于是将allocation1直接放置到老年代中，此时老年代中应该有4M的内存占用，之后垃圾收集结束。JVM直接在Eden空间给allocation2分配内存，此时Eden空间应该有4M的内存占用。查看GC日志可以知道Eden空间大约占用了4M内存，老年代大约占用了4M内存，与我们的预期一致。

```
[GC[DefNew: 5423K->376K(9216K), 0.0053000 secs] 5423K->4472K(19456K), 0.0053260 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
Heap
 def new generation   total 9216K, used 4723K [0x00000007f9a00000, 0x00000007fa400000, 0x00000007fa400000)
  eden space 8192K,  53% used [0x00000007f9a00000, 0x00000007f9e3ebc0, 0x00000007fa200000)
  from space 1024K,  36% used [0x00000007fa300000, 0x00000007fa35e388, 0x00000007fa400000)
  to   space 1024K,   0% used [0x00000007fa200000, 0x00000007fa200000, 0x00000007fa300000)
 tenured generation   total 10240K, used 4096K [0x00000007fa400000, 0x00000007fae00000, 0x00000007fae00000)
   the space 10240K,  40% used [0x00000007fa400000, 0x00000007fa800010, 0x00000007fa800200, 0x00000007fae00000)
 compacting perm gen  total 21248K, used 2930K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)
   the space 21248K,  13% used [0x00000007fae00000, 0x00000007fb0dcae0, 0x00000007fb0dcc00, 0x00000007fc2c0000)
No shared spaces configured.
```

NOTE：对于我们算出来的内存占用值有一些误差是正常的，因为我们只是算了占大多数的那部分，程序中还有一些静态变量等会占用一点堆内存。

```java
private static final int _1MB = 1024 * 1024;
public static void simpleGC2(){
    byte[] allocation1, allocation2, allocation3;
    allocation1 = new byte[_1MB];
    //什么时候进入老年代取决于XX:MaxTenuringThreshold设置
    allocation2 = new byte[4 * _1MB];
    allocation3 = new byte[4 * _1MB];  //这里产生一次Minor GC
    allocation3 = null;
    allocation3 = new byte[4 * _1MB];  //这里产生一次Minor GC
}
```

上面这个例子是在《深入理解Java虚拟机》中的一个例子，其中allocation3=null是不是有什么特殊含义呢？其实这一句根本没啥作用，有和没有没啥区别。

当程序执行到这一句`allocation3 = new byte[4 * _1MB];`的时候，Eden空间已经分配了5M内存，不够再分配4M内存了，此时会产生一次Minor GC，将allocation1和allocation2放到老年代里。GC执行完之后，将allcation3分配到Eden空间中，此时Eden空间占用4M。之后执行`allocation3=null`其实只是把对象指向的对象置为空而已。程序执行到`allocation3 = new byte[4 * _1MB];`的时候，发现Eden空间不够用了，再次促发一次Minor GC，此时allocation1、allocation2不会被回收，而allocation3会被回收。Minor GC结束之后，老年代占用5M，而JVM会给allocation3再在Eden区分配4M内存空间，因此Eden区占用4M。从GC日志中可以证明我们推论的正确。

所以说，其实`allocation3=null`这条语句没啥作用，如果要说作用，那么让内存少占用点空间吧，防止因为内存不足而溢出，或许《深入理解Java虚拟机》的作者就是这个意思？

GC日志：

```
[GC[DefNew: 6447K->377K(9216K), 0.0057550 secs] 6447K->5497K(19456K), 0.0057840 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC[DefNew: 4641K->357K(9216K), 0.0030110 secs] 9761K->5477K(19456K), 0.0030340 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 4647K [0x00000007f9a00000, 0x00000007fa400000, 0x00000007fa400000)
  eden space 8192K,  52% used [0x00000007f9a00000, 0x00000007f9e30660, 0x00000007fa200000)
  from space 1024K,  34% used [0x00000007fa200000, 0x00000007fa259780, 0x00000007fa300000)
  to   space 1024K,   0% used [0x00000007fa300000, 0x00000007fa300000, 0x00000007fa400000)
 tenured generation   total 10240K, used 5120K [0x00000007fa400000, 0x00000007fae00000, 0x00000007fae00000)
   the space 10240K,  50% used [0x00000007fa400000, 0x00000007fa900020, 0x00000007fa900200, 0x00000007fae00000)
 compacting perm gen  total 21248K, used 2931K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)
   the space 21248K,  13% used [0x00000007fae00000, 0x00000007fb0dcc00, 0x00000007fb0dcc00, 0x00000007fc2c0000)
No shared spaces configured.
```



## JVM参数总结

|参数名|含义|默认值|备注|
|---|---|---|---|
|-Xms|初始堆大小|物理内存的1/64(<1GB)|默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制.|
|-Xmx|最大堆大小|物理内存的1/4(<1GB)|默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制|
|-Xmn|年轻代大小(1.4or lator)||增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8|
|-XX:PermSize|设置持久代(perm gen)初始值|物理内存的1/64||
|-XX:MaxPermSize|设置持久代最大值|物理内存的1/4||
|-Xss|每个线程的堆栈大小|JDK5.0以后每个线程堆栈大小为1M|一般小的应用， 如果栈不是很深， 应该是128k够用的 大的应用建议使用256k。|
|-XX:NewRatio|年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代)||-XX:NewRatio=4表示年轻代与年老代所占比值为1:4,年轻代占整个堆栈的1/5|Xms=Xmx 并且设置了Xmn的情况下，该参数不需要进行设置。|
|-XX:SurvivorRatio|Eden区与Survivor区的大小比值|默认为8|设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10|
|-XX:MaxTenuringThreshold|垃圾最大年龄||如果设置为0的话,则年轻代对象不经过Survivor区,直接进入年老代. 对于年老代比较多的应用,可以提高效率.如果将此值设置为一个较大值,则年轻代对象会在Survivor区进行多次复制,这样可以增加对象再年轻代的存活 时间,增加在年轻代即被回收的概率该参数只有在串行GC时才有效.|

## 垃圾收集器参数总结

- **-XX:+PrintGC**：开启GC日志打印。默认不开启
- **-XX:-PrintGCDetails**：打印GC回收的细节。1.4.0引入，默认不启用。
- **XX:+PrintTenuringDistribution**：查看每次minor GC后新的存活周期的阈值

参考资料：   
1.[Java 6 JVM参数选项大全（中文版）](http://www.blogjava.net/bitbit/archive/2009/11/30/304247.html)
2.[http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html](http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html)
3.[http://www.cnblogs.com/redcreen/archive/2011/05/05/2038331.html](http://www.cnblogs.com/redcreen/archive/2011/05/05/2038331.html)


