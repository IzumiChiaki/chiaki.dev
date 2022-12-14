+++
title = "JVM垃圾回收"
date = "2020-08-10T22:53:19+08:00"
tags = ["jvm", "gc"]
slug = "jvm-gc"
dropCap = false
+++
#### 1. 判断对象是否可以被回收

- 引用计数法：每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，但**无法解决对象相互循环引用的问题**。

  ```java
  // 循环引用
  Node a = new Node();
  Node b = new Node();
  a.next = b;
  b.next = a;
  ```

- 可达性分析：从 GC Roots 开始向下搜索，搜索所走过的路径称为引用链。当一个对象到 GC Roots 没有任何引用链时，则证明此对象时不可用的，可以被回收。

  可以作为 GC Roots 的对象包括下面几种：

    - 虚拟机栈（栈帧中的本地变量表）中引用的对象；
    - 方法区中类静态属性引用的对象；
    - 方法区中常量引用的对象；
    - 本地方法栈中 JNI（即一般说的 Native 方法）引用的对象。

为了避免对象相互循环引用的问题，JVM 中使用可达性分析判断对象是否可以被回收。

#### 2. 四种引用

##### 2.1 强引用（StrongReference）

大部分引用都是强引用，是使用最普遍的引用。强引用不会被垃圾回收器回收，始终是可达状态。当内存空间不足时，JVM 宁可抛出 OOM 异常，使程序异常终止，也不会通过回收具有强引用的对象来解决内存不足的问题。这是造成 Java 内存泄漏的重要原因之一（联想 ThreadLocal 例子）。

##### 2.2 软引用（SoftReference）

对于具有软引用的对象，当内存空间足够时，垃圾回收器不会回收它，若内存空间不够了，就会回收这些对象的内存。软引用可以用来实现内存敏感的高速缓存。

##### 2.3 弱引用（WeakReference）

只要垃圾回收机制运行并发现了具有弱引用的对象，不管当前内存空间是否足够，都会回收该对象的内存。垃圾回收线程的优先级很低，不一定很快发现只具有弱引用的对象。

##### 2.4 虚引用（PhantomReference）

主要作用是用于跟踪对象被垃圾回收的活动。不能单独使用，必须与引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它具有虚引用，就会在回收对象的内存前把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入虚引用来了解被引用的对象是否要被垃圾回收。

#### 3. 判断废弃常量和无用的类

**废弃常量**：运行时常量池主要回收废弃常量。如果当前没有任何对象引用该常量，就说明该常量是废弃常量。

**无用的类**：需要同时满足3个条件。

- 该类的所有实例都已经被回收，即堆中不存在该类的任何实例。
- 加载该类的 ClassLoader 已经被回收
- 该类对于的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

#### 4. 垃圾收集算法

##### 4.1 标记-清除算法（Mark-Sweep）

**原理**：首先标记出可以回收的对象，在标记完成后统一回收所有被标记的对象。

![mark-sweep.png](/images/jvm-gc/mark-sweep.png)

**特点**：**内存碎片化严重、效率低**。

##### 4.2 复制算法（Copying）

**原理**：按内存容量将内存划分为大小相同的两块。每次只使用其中的一块，当这一块内存满了之后，将存活的对象复制到未使用的一块中，然后把使用的那一块一次清理。

![copying.png](/images/jvm-gc/copying.png)

**特点：不易产生内存随便，但效率大大降低。**

##### 4.3 标记-整理算法（Mark-Compact）

**原理**：首先标记出可以回收的对象，然后将存活对象移动至内存的一端，然后清除掉边界外的对象。

![mark-compact.png](/images/jvm-gc/mark-compact.png)

**特点：根据老年代特点提出的一种标记算法。**

##### 4.4 分代收集算法

当前虚拟机的垃圾收集都采用分代收集算法。

在..新生代..，每次收集都会有大量对象死去，所以采用**复制算法**，只需复出少了对象的复制成本就可以完成每次垃圾收集。

在..老年代..，对象存活几率比较高，每次垃圾回收是时只有少量对象需要被回收，因此采用**标记-整理算法**进行垃圾收集。

#### 5. 垃圾收集器

JVM 针对新生代和老年代分别提供了不同的垃圾收集器。

![collectors.png](/images/jvm-gc/collectors.png)

##### 5.1 Serial收集器（单线程+复制算法）

Serial 收集器是最基本的垃圾收集器。Serial 是单线程的收集器，只会使用一条垃圾收集线程去完成垃圾收集工作，同时在进行垃圾收集工作时不许暂停其他所有的工作线程（"Stop the World"），直到收集结束。

新生代采用复制算法，老年代采用标记-整理算法。

Serial 收集器简单高效，对于限定单个 CPU 环境来说没有线程交互的开销，可以获得最高的单线程垃圾收集效率，因此 Serial 垃圾收集器时 JVM 运行在 Client 模式下默认的新生代垃圾收集器。

![serial.png](/images/jvm-gc/serial.png)

##### 5.2 ParNew收集器（Serial+多线程）

ParNew收集器是Serial收集器的多线程版本（多线程并发），除了使用多线程进行垃圾收集外，其余行为和 Serial 收集器完全一样。

新生代采用复制算法，老年代采用标记-整理算法。

ParNew收集器是运行在 Server 模式下的 JVM 的首要选择，除了 Serial 收集器，只有它能与 CMS 收集器配合工作。

![ParNew.png](/images/jvm-gc/ParNew.png)

##### 5.3 Parallel Scavenge收集器

与 ParNew 收集器类似，但 Parallel Scavenge 收集器的关注点是吞吐量（高效率地利用 CPU ）。CMS 等垃圾收集器地关注点是用户线程的停顿时间（提高用户体验）。所谓吞吐量就是 CPU 中用于运行用户代码的时间与 CPU 总消耗时间的比值。

Parallel Scavenge 提供了两个参数用于精确控制吞吐量，分别是控制最大垃圾收集停顿时间的`-XX: MaxGCPauseMillis`参数以及直接设置吞吐量大小的`-XX: GCTimeRatio`参数。

##### 5.4 Serial Old收集器（单线程+标记-整理算法）

Serial 收集器的老年代版本。它具有两大用途：一种是在 JDK1.5 及以前的版本中与 Parallel Scavenge 收集器搭配使用，另一种是作为 CMS 收集器的后备方案。

##### 5.5 Parallel Old收集器（多线程+标记-整理算法）

Parallel Scavenge 收集器的老年代版本，使用多线程和标记-整理算法。在注重吞吐量和 CPU 资源的场合，可以优先考虑 Parallel Scavenge 收集器和 Parallel Old 收集器。

##### 5.6 CMS收集器（多线程+标记-清除算法）

CMS（Concurrent Mark Sweep）收集器是一种**以获取最短回收停顿时间为目标**的收集器，非常符合在注重用户体验的应用上使用。 CMS 收集器是第一款真正意义上的并发收集器，第一次实现了让垃圾回收线程与用户线程（基本上）同时工作。CMS 收集器是基于**标记-清除算法**的。其运行过程包括四个步骤：

![concurrent-mark-sweep.png](/images/jvm-gc/concurrent-mark-sweep.png)

- **初始标记**：**暂停所有其他线程**，并记录下直接与 GC Roots 相连的对象，速度很快；
- **并发标记**：同时开启 GC 和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断地更新引用域，所以 GC 线程无法保证可达性分析的实时性，因此这个算法里会跟踪记录这些发生引用更新的地方。该阶段**无需暂停工作线程**；
- **重新标记**：重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记近阶段的时间短。该阶段仍需要**暂停所有工作线程**；
- **并发清除**：开启用户线程，同时GC线程开始清除 GC Roots 不可达对象，**无需暂停工作线程**。

优点：**并发收集、低停顿**。

缺点：对 CPU 资源敏感；无法处理浮动垃圾；使用的"标记-清除"会导致收集结束时产生大量内存碎片。

##### 5.7 G1收集器

G1（Garbage First）是面向服务器的垃圾收集器，主要针对配备多核处理器及大容量内存的机器，**以极高概率满足 GC 停顿时间要求的同时还具备高吞吐量性能特征**。

特点：

- **并行与并发**
- **分代收集**：可以独立管理整个 GC 堆，但也保留了分代的概念；
- **空间整合**：基于标记-整理算法，不产生内存碎片；
- **可预测的停顿**：相比于 CMS，G1 可以能建立可预测的停顿时间模型，在不牺牲吞吐量的前提下实现低停顿的垃圾回收。

G1 收集器运行步骤大致分为：**初始标记；并发标记；最终标记；筛选回收**。

G1 收集器避免全区域垃圾收集，把堆内存划分为大小固定的几个独立区域，并且跟踪这些区域的垃圾收集进度，同时在后台维护一个优先级列表，每次根据所允许的收集时间，优先回收垃圾最多的区域。区域划分和优先级区域回收机制，确保 G1 收集器可以在有限时间获得最高的垃圾收集效率。