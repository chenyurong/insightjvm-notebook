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


