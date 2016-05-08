# 垃圾回收器

## 新生代垃圾回收器

### Serial

单线程

### ParNew

多线程版本的Serial

### Parallel Scavenge 

多线程。以达到可控制的吞吐量为目标

## 老年代垃圾回收器

### Serial Old 

Serial垃圾回收器的老年代版本

### Parallel Old

Parallel Scavenge 垃圾回收期的老年代版本

### CMS

以最短停顿时间为目标的垃圾回收器

## G1垃圾回收器

最先进的垃圾回收器