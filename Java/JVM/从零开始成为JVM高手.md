# JVM实战学习笔记


| 参数                               | 用途                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| -Xms                               | 初始堆大小                                                   |
| -Xmx                               | 最大堆大小                                                   |
| -Xmn                               | 年轻代大小                                                   |
| –XX:NewRatio                       | 新生代和老年代的比例，–XX:NewRatio=2，表示新生代占整个堆的1/3，老年代占堆2/3 |
| -XX:SurvivorRatio                  | 年轻代中eden区和单个survivor区的大小比值，默认为8，eden区占年轻代8/10 |
| -XX:+UseParNewGC                   | 指定新生代的垃圾回收器使用ParNew                             |
| -XX:ParallelGCThreads              | 调节ParNew的垃圾回收线程数量                                 |
| -server                            | 以服务器模式启动                                             |
| -client                            | 以客户端模式启动                                             |
| -XX:CMSInitiatingOccupancyFaction  | 设置老年代占用多少比例的时候触发CMS垃圾回收，JDK6默认值92%   |
| -XX:+UseCMSCompactAtfullCollection | 表示full gc之后会进行stop the world，停止工作线程，进行碎片整理，**默认开启** |
| -XX:CMSfullGCsBeforeCompaction     | 标识执行多少次full gc后再执行一次内存碎片的整理工作，**默认值为0**，表示每次full gc之后都会进行一次内存碎片的整理工作 |
| -XX:-HandlePromotionFailure | 空间担保机制，在Minor GC触发之前，发现老年代的可用内存小于新生代的全部对象的大小，会根据这个参数有没有设置去判断是否需要触发 Full GC，如果没有设置这个参数，会直接触发一次Full GC，如果设置了这个参数，会判断老年代的内存是否大于历史Minor GC后进入老年代的平均大小，如果是，则表示可以冒险进行Minor GC，如果不是，则会触发一次Full GC|
| -XX:+UseG1GC | 指定使用G1垃圾回收器 |
| -XX:G1HeapRegionSize | 使用G1垃圾回收器的时候，指定堆中的region的数量，必须是**2的n次幂** |                                                         |
| -XX:G1NewSizePrecent | 使用G1垃圾回收器的时候，设置堆中**新生代的初始占比**，**默认值为5%** |
| -XX:G1MaxNewSizePrecent | 使用G1垃圾回收器的时候，设置堆中**新生代的最大占比**，**默认值为60%** |
| -XX:MaxGcPauseMills | 设置G1在执行GC的时候最多可以让系统停顿多长时间，**默认值200ms** |
| -XX:MaxTenuringThreshold | 设置对象从新生代进入老年代需要经历的Minor GC次数 |
| -XX:InitiatingHeapOccupancyPrecent | 设置触发标记周期的 Java 堆占用率阈值，默认占用率是整个 Java 堆的 45% |
| -XX:G1MixedGCCountTarget | 表示G1垃圾回收器在一次混合回收过程中，最后一个阶段执行几次混合回收，**默认为8次** |
| -XX:G1HeapWastePercent | 表示G1垃圾回收过程中，清理后腾空的region占对内存的一定百分比后，停止垃圾回收，**默认是5%** |
| -XX:G1MixedGCLiveThresholdPrecent | 表示一个region中对象存活的比例，当低于**默认值85%**的时候，这个region才可以进行垃圾回收 |
| -XX:+CMSParallelInitialMarkEnabled | 表示在CMS垃圾回收器的“初始标记”阶段开启多线程并发执行 |
| -XX:+CMSScavengeBeforeRemark | 表示在CSM“重新标记”阶段，先尽量执行一次Young GC（可以少扫描一些对象） |
| -XX:+DisableExplicitGC | 表示**禁止显示执行GC** |
| -XX:MetaspaceSize | 设置元空间大小 |
| -XX:MaxMetaspaceSize | 设置最大元空间大小 |
| -XX:ThreadStackSize | 设置栈内存 |




服务器模式和客户端模式的区别：

- 服务器模式通常运行网站、电商、业务、APP后台等大型系统，一般是多核CPU（合适ParNew垃圾回收器）

- 客户端程序，例如百度云盘windows客户端之类的，通常是单核CPU（合适Serial垃圾回收器）


Minor GC 复制算法

CMS垃圾回收器的垃圾回收算法：

- 标记清理算法

CMS通过垃圾回收线程和系统工作线程尽量同时执行的模式来进行垃圾回收



CMS垃圾回收四个阶段：

- 初始标记
stop the world，标记出所有GC Roots**直接引用**的对象，影响不大，**速度快**

- 并发标记
系统线程可以继续运行，随意创建新对象，对老年代所有对象进行**GC Roots追踪（最耗时）**

- 重新标记
stop the world，重新标记第二阶段新建的对象，**速度快**

- 并发清理
系统恢复运行，垃圾回收器清理掉之前标记出的垃圾对象（耗时）


CMS默认启动的垃圾回收线程的数量是(CPU +3)/4

CMS并发垃圾回收的问题

1. 消耗CPU资源（CPU资源一部分被垃圾回收线程占用）

1. concurrent mode failure问题
在“并发清理”阶段，CMS只是会清理之前标记好的垃圾对象，过程中仍然会有对象进入老年代（浮动垃圾），需要等待下一次GC才能回收他们
在full gc的过程中，进入老年代的垃圾对象大于老年代中的空闲空间，就会出现concurrent mode failure问题，并发垃圾回收失败
此时会**自动使用“serial full”垃圾回收器替代CMS**，强行stop the world，重新标记垃圾对象（过程中不允许新对象产生）

1. “标记 - 清理”算法导致大量内存碎片的产生
内存碎片太多，导致进入老年代的垃圾对象**找不到连续可用的空间**，会触发full gc


CMS的触发时机：
    老年代的内存占用到达一定比例

Full GC比Minor GC慢很多的原因：

- 新生代存活的对象少，通过GC Roots标记出这些对象后，直接丢到survivor区，然后直接对eden区和另一个survivor区进行垃圾回收即可

- 老年代存活对象多，标记工程耗时长

- 老年代进行垃圾回收的不是一大片连续的内存区域，而是很多的不连续，零散的内存区域（所以会造成很多内存碎片）

- 老年代垃圾回收完毕以后，还要执行一次内存碎片的整理工作

- 老年代垃圾回收过程中可能触发concurrent mode failure，这样会导致垃圾回收工作从头开始（真TM蛋疼）


Full GC的触发时机：

- 在没有开启空间担保的情况下，触发Minor GC之前，发现老年代的可用内存小于新生代的全部对象的大小，则触发一次full gc

- 在开启了空间担保的情况下，触发Minor GC之前，发现历史Minor GC后进入老年代的平均对象大小大于老年代的可用内存，则触发一次full gc

- 老年代的内存占用率到达一定程度的时候，会触发full gc


对象从新生代**进入老年代**的条件（满足其中任意一条）：

- Minor GC标记存活的对象大于survivor区

- 动态年龄判断

- 对象经历了15次（可通过参数设置）的Minor GC



# G1垃圾回收器

- 可以**同时回收**新生代和老年代的对象，不需要两个垃圾回收器

- 将JVM内存拆分为**多个大小相等**的region（新生代和老年代的逻辑划分）

- 可以设置一个垃圾回收的预期停顿时间


region的回收价值：region中可以回收的对象大小和回收的预估时间



## 核心设计思路

​    将堆拆分成大量region，通过追踪每个region中可以回收的对象大小和预估时间，尽量把垃圾回收对系统造成的影响控制在指定的时间范围内，**在有限的时间尽量回收尽可能多的垃圾对象**

**region随时会属于新生代或老年代**



region的数量与大小：

- JVM中最多可以有2048个region（region的数量必须是2的倍数）

- region的大小 = 堆大小 / region的数量

- 可以通过参数“-XX:G1HeapRegionSize”指定region的数量



堆中的新生代占比：·

- 默认堆中的新生代占比是5%

- JVM运行过程中会不断提高堆中的新生代的占比，占比最多60%，可以通过参数“-XX:G1MaxNewSizePrecent”设置



region大对象的存放：

- G1提供了专门的region存放大对象，而不是让大对象直接进入老年代的region中

- 对象超过一个region的50%就算是大对象

- 如果一个对象太大，会横跨多个region存放

G1垃圾回收器触发新生代+老年代混合垃圾回收的时机：
    如果老年代占据了45%的region，此时会尝试触发一个新生代+老年代混合垃圾回收的阶段（可以通过参数“-XX:InitiatingHeapOccupancyPrecent”参数配置）GG

混合垃圾回收过程跟老年代的过程差不多

G1垃圾回收器允许执行多次混合垃圾回收（在垃圾回收的最后一个阶段），由参数“-XX:G1MixedGCCountTarget”参数控制，默认八次

**混合垃圾回收是基于复制算法进行的，**把要回收的region中的存活对象放入其他region中，然后将这个region清理掉，这样在回收过程会不断腾出空闲的region，一旦空闲的region数量达到了对队内存的5%（由参数“-XX:G1HeapWastePercent”），就会停止混合回收。

G1整体是基于复制算法进行垃圾回收，所以不会像CMS那样垃圾回收后出现内存碎片问题，但是当在复制过程中，没有发现有其余空闲的region可以接收存活的对象，会进入stop the world，采用单线程标记，清理，压缩整理，腾出一批空闲的region（这个过程非常慢）

回收region的时候，**要求region中的存活对象低于85%**，由参数“-XX:G1MixedGCLiveThresholdPrecent”决定

混合垃圾回收的优化思路：

- 尽量避免对象过快进入老年代

- 合理设置“-XX:MaxGcPauseMills”参数


# 线上JVM参数配置
```shell
-server
-Xms3072m
-Xmx3072m
-Xss300k
-XX:MetaspaceSize=256m
-XX:NewRatio=2
-XX:+UseConcMarkSweepGC
-XX:+UseParNewGC  
-XX:+UseCMSCompactAtFullCollection  
-XX:CMSFullGCsBeforeCompaction=1
-XX:CMSInitiatingOccupancyFraction=75
-XX:+CMSClassUnloadingEnabled
-XX:+DisableExplicitGC
-verbose:gc
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-Xloggc:/home/logs/crm-web/gc.log
-XX:+HeapDumpBeforeFullGC
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/mfsdata/javaHeapDump/crm-web/
-XX:+PrintClassHistogramBeforeFullGC
-XX:+PrintClassHistogramAfterFullGC
-XX:ErrorFile=/home/logs/crm-web/hs_err_pid%p.log
-Djava.endorsed.dirs=/usr/local/tomcat/endorsed
```





# 一套通用的JVM参数模板
![image.png](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/1577084011736-13829090-1f88-4db9-8fcf-dd2deb038fc3.png)



# 设置合理的JVM参数的原则

- 尽可能在每次young gc后，留下来的存活对象远小于survivor区
- 最理想的状态是系统几乎不发生full gc，老年代稳定占用一定空间



# 线上频繁full gc的几种表现

- 机器CPU负载过高
- 频繁full gc告警
- 系统无法处理请求或处理过慢



# 频繁full gc的几种常见的原因

- 系统承载高并发请求
- 处理数据量过大，导致频繁young gc
- 内存分配不合理，survivor区过小，导致对象频繁进入老年代，频繁触发full gc
- 系统一次性加载数据过多，产生很多大对象，导致频繁有大对象进入老年代，频繁触发full gc
- 系统发生内存泄漏，莫名奇妙创建大量对象并无法回收，一直占用老年代
- 元空间加载类过多，导致频繁full gc
- 错误调用System.gc()




![image.png](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/1577084011778-1cac14e6-5391-408a-a9c8-85ed1030161c.png)



# Metaspace内存溢出的原因（一般来说）

- 生产环境直接使用Metaspace默认参数

- 代码中通过动态代理之类的技术，产生过多的类，把Metaspace填满



# 服务出现接口假死情况：

- 接口占用大量内存，无法释放，频繁gc，频繁stop the world

- cpu负载过高，某个进程耗尽CPU资源