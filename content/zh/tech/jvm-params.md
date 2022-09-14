+++
title = "JVM参数"
date = "2020-08-11T00:43:19+08:00"
tags = ["jvm"]
slug = "jvm-params"
dropCap = false
+++

#### 1. JVM常用参数

| **配置项**                  | **例子**                     | **含义**                                        | **备注**                                                                                                                                     |
|--------------------------|----------------------------|-----------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| -Xmx                     | -Xmx20m                    | java 应用最大可用内存为 20M                            | **整个 JVM 内存大小=年轻代大小 + 年老代大小 + 持久代大小**。持久代一般固定大小为 64M，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun 官方推荐配置为整个堆的 3/8。                                   |
| -Xms                     | -Xms5m                     | java 应用最小分配内存为 5M                             |                                                                                                                                            |
| -Xmn                     | -Xmn1m                     | 新生代配置的内存为 1M                                  |                                                                                                                                            |
| -XX:NewRatio             | -XX:NewRatio=3             | 新生代和年老代的内存分配比例为1:4                            | **-XX:NewRatio=4**:设置年轻代（包括 Eden 和两个 Survivor 区）与年老代的比值（除去持久代）。设置为 4，则年轻代与年老代所占比值为 1:4，年轻代占整个堆栈的 1/5                                       |
| -XX:SurvivorRatio        | -XX:SurvivorRatio=8        | 新生代内部 Survivor 和 Eden 区域的比例为Survivor：Eden=2:8 | **-XX:SurvivorRatio=4**：设置年轻代中 Eden 区与 Survivor 区的大小比值。设置为 4 ，则两个 Survivor 区与一个 Eden 区的比值为 2:4，一个 Survivor 区占整个年轻代的 1/6                    |
| -Xss                     | -Xss128k                   | 设置每个线程的堆栈大小                                   | JDK5.0 以后每个线程堆栈大小为 1M，以前每个线程堆栈大小为 256K。根据应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在 3000~5000 左右。           |
| -XX:MaxPermSize          | -XX:MaxPermSize=16m        | 设置持久代大小为 16M。                                 |                                                                                                                                            |
| -XX:MaxTenuringThreshold | -XX:MaxTenuringThreshold=0 | 设置垃圾最大年龄                                      | **如果设置为 0 的话，则年轻代对象不经过 Survivor 区，直接进入年老代**对于年老代比较多的应用，可以提高效率。**如果将此值设置为一个较大值，则年轻代对象会在 Survivor 区进行多次复制，这样可以增加对象再年轻代的存活时间**，增加在年轻代即被回收的概论。 |

#### 2. 垃圾收集器相关参数

##### 2.1 串行收集器

`-XX:+UseSerialGC`：设置串行收集器

##### 2.2 并行收集器（吞吐量优先）

`-XX:+UseParallelGC`：设置为并行收集器。此配置仅对年轻代有效，即年轻代使用并行收集而年老代仍使用串行收集。

`-XX:ParallelGCThreads=20`：配置并行收集器的线程数，即：同时有多少个线程一起进行垃圾回收。此值建议配置与 CPU 数目相等。

`-XX:+UseParallelOldGC`：配置年老代垃圾收集方式为并行收集。JDK6.0 开始支持对年老代并行收集。

`-XX:MaxGCPauseMillis=100`：设置每次年轻代垃圾回收的最长时间（单位毫秒）。如果无法满足此时间，JVM 会自动调整年轻代大小，以满足此时间。

`-XX:+UseAdaptiveSizePolicy`：设置此选项后，并行收集器会自动调整年轻代 Eden 区大小和 Survivor 区大小的比例，以达成目标系统规定的最低响应时间或者收集频率等指标。此参数建议在使用并行收集器时，一直打开。

##### 2.3 并发收集器（响应时间优先）

`-XX:+UseConcMarkSweepGC`：即 CMS 收集，设置年老代为并发收集。CMS 收集是 JDK1.4 后期版本开始引入的新 GC 算法。它的主要适合场景是对响应时间的重要性需求大于对吞吐量的需求，能够承受垃圾回收线程和应用线程共享CPU资源，并且应用中存在比较多的长生命周期对象。CMS 收集的目标是尽量减少应用的暂停时间，减少 Full GC 发生的几率，利用和应用程序线程并发的垃圾回收线程来标记清除年老代内存。

`-XX:+UseParNewGC`：设置年轻代为并发收集。可与 CMS 收集同时使用。JDK5.0 以上，JVM 会根据系统配置自行设置，所以无需再设置此参数。

`-XX:CMSFullGCsBeforeCompaction=0`：由于并发收集器不对内存空间进行压缩和整理，所以运行一段时间并行收集以后会产生内存碎片，内存使用效率降低。此参数设置运行 0 次 Full GC 后对内存空间进行压缩和整理，即每次 Full GC 后立刻开始压缩和整理内存。

`-XX:+UseCMSCompactAtFullCollection`：打开内存空间的压缩和整理，在 Full GC 后执行。可能会影响性能，但可以消除内存碎片。

`-XX:+CMSIncrementalMode`：设置为增量收集模式。一般适用于单 CPU 情况。

`-XX:CMSInitiatingOccupancyFraction=70`：表示年老代内存空间使用到70%时就开始执行 CMS 收集，以确保年老代有足够的空间接纳来自年轻代的对象，避免 Full GC 的发生。

##### 2.4 其它垃圾回收参数

`-XX:+ScavengeBeforeFullGC`：年轻代 GC 优于 Full GC 执行。

`-XX:-DisableExplicitGC`：不响应 System.gc() 代码。

`-XX:+UseThreadPriorities`：启用本地线程优先级 API。即使 java.lang.Thread.setPriority() 生效，不启用则无效。

`-XX:SoftRefLRUPolicyMSPerMB=0`：软引用对象在最后一次被访问后能存活 0 毫秒（JVM默认为 1000 毫秒）。

`-XX:TargetSurvivorRatio=90`：允许 90% 的 Survivor 区被占用（JVM默认为 50% ）。提高对于 Survivor 区的使用率。

##### 2.5 辅助信息参数设置

`-XX:-CITime`：打印消耗在JIT编译的时间。

`-XX:ErrorFile=./hs_err_pid.log`：保存错误日志或数据到指定文件中。

`-XX:HeapDumpPath=./java_pid.hprof`：指定 Dump 堆内存时的路径。

`-XX:-HeapDumpOnOutOfMemoryError`：当首次遭遇内存溢出时 Dump 出此时的堆内存。

`-XX:OnError=";"`：出现致命 ERROR 后运行自定义命令。

`-XX:OnOutOfMemoryError=";"`：当首次遭遇内存溢出时执行自定义命令。

`-XX:-PrintClassHistogram`：按下 Ctrl+Break 后打印堆内存中类实例的柱状信息，同 JDK 的 jmap -histo 命令。

`-XX:-PrintConcurrentLocks`：按下 Ctrl+Break 后打印线程栈中并发锁的相关信息，同 JDK 的 jstack -l 命令。

`-XX:-PrintCompilation`：当一个方法被编译时打印相关信息。

`-XX:-PrintGC`：每次 GC 时打印相关信息。

`-XX:-PrintGCDetails`：每次 GC 时打印详细信息。

`-XX:-PrintGCTimeStamps`：打印每次 GC 的时间戳。

`-XX:-TraceClassLoading`：跟踪类的加载信息。

`-XX:-TraceClassLoadingPreorder`：跟踪被引用到的所有类的加载信息。

`-XX:-TraceClassResolution`：跟踪常量池。

`-XX:-TraceClassUnloading`：跟踪类的卸载信息。

##### 2.6 其它更多配置

`-XXThreadStackSize`：设置内存页的大小，不可设置过大，会影响 Perm 的大小。

`-XX:+UseFastAccessorMethods`：设置原始类型的快速优化。

`-XX:+DisableExplicitGC`：设置关闭 System.gc() (这个参数需要严格的测试)。

`-XX:MaxTenuringThreshold`：设置垃圾最大年龄。如果设置为0的话,则年轻代对象不经过 Survivor 区，直接进入年老代。 对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在 Survivor 区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概率。该参数只有在串行 GC 时才有效。

`-XX:+AggressiveOpts`：加快编译。

`-XX:+UseBiasedLocking`：锁机制的性能改善。

`-Xnoclassgc`：禁用垃圾回收。

`-XX:SoftRefLRUPolicyMSPerMB`：设置每兆堆空闲空间中 SoftReference 的存活时间，默认值是 1s 。

`-XX:PretenureSizeThreshold`：设置对象超过多大时直接在旧生代分配，默认值是 0。

`-XX:TLABWasteTargetPercent`：设置 TLAB 占 eden 区的百分比，默认值是 1% 。

`-XX:+CollectGen0First`：设置 Full GC 时是否先 YGC，默认值是 false。