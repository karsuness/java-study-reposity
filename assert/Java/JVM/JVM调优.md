# JVM 调优

> 来源：`JVM调优.md`

## 什么时候需要进行JVM调优

  * Heap内存（老年代）持续上涨达到设置的最大内存值
  * Full GC次数频繁
  * GC 停顿时间过长（超过1秒）
  * 应用出现OutOfMemory等内存异常
  * 应用中有使用本地缓存且占用大量内存空间
  * 系统吞吐量与响应性能不高或下降（系统吞吐量：系统在单位时间内所处理的信息量）

## JVM调优基本原则

  * 大多数Java应用不需要进行JVM优化
  * 大多数导致GC问题的原因是代码层面导致的
  * 上线之前，应先考虑将机器的JVM参数设置到最优
  * 减少创建对象的数量（代码层面）
  * 减少使用全局变量和大对象（代码层面）
  * 优化架构调优和代码调优，JVM是不得已的手段（代码、架构层面）
  * 分析GC情况优化代码比优化JVM参数更好（代码层面）

## JVM调优目标

调优的最终目的都是为了令应用程序使用最小的硬件消耗来承载更大的吞吐。JVM调优主要是针对垃圾收集器的收集性能优化，令运行在虚拟机上的应用能够使用更少的内存以及延迟获取更大的吞吐量

  * 延迟：GC低停顿和GC低频率
  * 低内存占用
  * 高吞吐量

其中，任何一个属性性能的提升，几乎都是以牺牲其他其他属性性能的损失为代价，不可多得，具体根据在业务中的重要性确定

## JVM调优的步骤

  1. 分析GC日志及dump文件，判断是否需要优化，确定瓶颈问题
  2. 确定JVM调优量化目标
  3. 确定JVM条有参数（根据历史JVM参数来调整）
  4. 依次调优内存、延迟、吞吐量等指标
  5. 对比观察调优前后的差异
  6. 不断的分析和调整，直到直到合适的JVM参数配置
  7. 找到最合适的参数，将这些参数应用到服务器，并进行后续跟踪

## JVM参数

-XX 参数被称为不稳定参数，此类参数的设置很容易引起JVM 性能上的差异，使JVM存在极大的不稳定性。如果此类参数设置合理将大大提高JVM的性能及稳定性。

不稳定参数语法规则包含以下内容。

布尔类型参数值：

-XX:+ '+'表示启用该选项

-XX:- '-'表示关闭该选项

数字类型参数值：

-XX:=给选项设置一个数字类型值，可跟随单位，例如：'m'或'M'表示兆字节;'k'或'K'千字节;'g'或'G'千兆字节。32K与32768是相同大小的。

字符串类型参数值：

-XX:=给选项设置一个字符串类型值，通常用于指定一个文件、路径或一系列命令列表。例如：-XX:HeapDumpPath=./dump.core

## JVM参数解析及调优

```java
-Xmx4g –Xms4g –Xmn1200m –Xss512k -XX:SurvivorRatio=8 -XX:PermSize=100m -XX:MaxPermSize=256m -XX:MaxTenuringThreshold=15

> 注：`-Xmn` 与 `-XX:NewRatio` 不能同时使用——两者都在指定新生代大小，同时出现时 **`-Xmn` 生效、`NewRatio` 被忽略**，所以这里只保留 `-Xmn`。
```

注意：Java8已经移除了永久代，-XX:PermSize=100m -XX:MaxPermSize=256m这两个配置已经无效了

参数解析：

  * **-Xmx4g**： 堆内存最大值为4GB。
  * **-Xms4g**： 初始化堆内存大小为4GB。
  * **-Xmn1200m：** 设置新生代大小为1200MB。增大新生代后，将会减小年老代大小。此值对系统性能影响较大。经验上新生代约占整个堆的 1/3 ~ 3/8（默认 `NewRatio=2` 即新生代占 1/3），但并没有官方的固定推荐值，需要结合对象的存活特性来调。
  * **-Xss512k：** 设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1MB，以前每个线程堆栈大小为256K。应根据应用线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。
  * **-XX:NewRatio=4** ：设置新生代（包括Eden和两个Survivor区）与年老代的比值（除去持久代）。设置为4，则年轻代与年老代所占比值为1：4，年轻代占整个堆栈的1/5
  * **-XX:SurvivorRatio=8：** 设置年轻代中Eden区与Survivor区的大小比值。设置为8，则两个Survivor区与一个Eden区的比值为2:8，一个Survivor区占整个年轻代的1/10
  * **-XX:PermSize=100m：** 初始化永久代大小为100MB。
  * **-XX:MaxPermSize=256m：** 设置永久代大小为256MB。
  * **-XX:MaxTenuringThreshold=15：** 设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概论。

```java
Heap Configuration:
   MinHeapFreeRatio         = 0                   //空闲堆空间的最小百分比,默认值为 40
   MaxHeapFreeRatio         = 100                 //空闲堆空间的最大百分比,默认值为 70
   MaxHeapSize              = 536870912 (512.0MB)   //JVM 堆空间允许的最大值
   NewSize                  = 178782208 (170.5MB)   //JVM 新生代堆空间的默认值
   MaxNewSize               = 178782208 (170.5MB)   //JVM 新生代堆空间允许的最大值
   OldSize                  = 358088704 (341.5MB)   //JVM 老年代堆空间的默认值
   NewRatio                 = 2                    //新生代（2个Survivor区和Eden区 ）与老年代（不包括永久区）的堆空间比值，表示新生代：老年代=1：2
   SurvivorRatio            = 8                    //两个Survivor区和Eden区的堆空间比值为 8，表示 S0 ： S1 ：Eden = 1：1：8
   MetaspaceSize            = 21807104 (20.796875MB) //JVM 元空间的默认值
   CompressedClassSpaceSize = 1073741824 (1024.0MB) //压缩类空间大小
   MaxMetaspaceSize         = 17592186044415 MB //JVM 元空间允许的最大值
```

### 可调优参数

  * -Xms：初始化堆内存大小，默认为物理内存的1/64(小于1GB)。
  * -Xmx：堆内存最大值。默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到-Xms的最小限制。
  * -Xmn：新生代大小，包括Eden区与2个Survivor区。
  * -XX:SurvivorRatio=1：Eden区与一个Survivor区比值为1:1。
  * -XX:MaxDirectMemorySize=1G：直接内存。报java.lang.OutOfMemoryError: Direct buffer memory异常可以上调这个值。
  * -XX:+DisableExplicitGC：让运行期显式调用 `System.gc()` 变成空操作（不再触发 Full GC）。注意它同时会影响 NIO 直接内存的回收——`DirectByteBuffer` 依赖 `System.gc()` 触发 Cleaner，关掉后可能出现直接内存堆积。

注意：Java RMI 的定时 GC 触发间隔可用 `-Dsun.rmi.dgc.server.gcInterval` 控制，**单位是毫秒**。默认约 1 小时（3600000）；写成 86400 实际只有 86.4 秒，会明显增加 Full GC 频率。

  * -XX:CMSInitiatingOccupancyFraction=60：老年代内存回收阈值，默认值为68。
  * -XX:ParallelCMSThreads=4：CMS垃圾回收器并发线程数，推荐值为CPU核心数。
  * -XX:ParallelGCThreads=8：新生代并行收集器的线程数。
  * -XX:MaxTenuringThreshold=10：设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概论。
  * -XX:CMSFullGCsBeforeCompaction=4：指定进行多少次fullGC之后，进行tenured区 内存空间压缩。
  * -XX:CMSMaxAbortablePrecleanTime=500：当abortable-preclean预清理阶段执行达到这个时间时就会结束。

在设置的时候，如果关注性能开销的话，应尽量把永久代的初始值与最大值设置为同一值，因为永久代的大小调整需要进行FullGC才能实现。

步骤

1.使用jps查看进程id

2.jmap -dump:format=b,file=dumpFileName pid 将内存信息dump到文件中 使用mat或者visualvm工具进行查看
