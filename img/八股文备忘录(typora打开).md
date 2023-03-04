## Java 基础

#### Java 异常体系

Error

> StackOverFlowError：一个线程栈的调用深度太深容易导致出现该问题，如递归方法，反复调用自身，栈的深度导致该异常。
>
> OutOfMemoryError: 内存溢出，常常发生在大对象申请导致内存不够，或者内存泄露导致可用内存不够分配的情况。

RuntimeException

> NullPointerException
>
> ArrayIndexOutOfBoundsException
>
> IllegalArgumentException

![image-20220427150854671](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220501212650.png)



#### Java 的不可变类

String 类和基本类型的包装类，都在元空间的常量池中：

​	举例 String 的实现就是 字符数组，其修饰是 private + final 的，属于不可变的类型。

![image-20220512172333934](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220512172341.png)





#### 代理和反射

参考：https://blog.csdn.net/qq_33945246/article/details/104757997

##### 反射的原理

- java 文件通过编译成为 class 字节码文件，生成到 MetaSpace 的 Class 常量池中
- 通过 Class.forName 类似的方式，按照类加载器的委托机制去加载对应 Class 对象
- 通过 Class 对象获取对应的构造函数、成员变量和方法等。 



## JVM 相关

### 字节码技术

Java 是一种面向对象、静态类型、编译执行，有 **VM/GC 和运行时**、跨平台的高级语言。

运行时：JVM运行一个java程序，需要启动的java进程以及依赖的库组成的环境。

**Java 代码通过编译成字节码，Java进程通过虚拟机内部的类加载器去编译字节码文件，生成对应的实例，运行其方法。其中三个重要的技术：字节码技术、类加载器和虚拟机内存管理。**

**字节码：由单字节（byte）的指令组成，理论上最多支持256个操作码**（opcode），实际上 Java 只使用了 200 左右的操作码，还有一些操作码保留作调试操作。

根据指令性质，**字节码主要分为四个大类**：

1. **栈操作指令，包括与局部变量交互的指令。**
2. **程序流程控制指令**
3. **对象操作指令，包括方法调用指令**
4. **算术运算以及类型转换指令**

javac 编译java文件 -g:加入额外的debug信息，能看到本地变量表，**javap -c -verbose class文件 反汇编 可以看到字节码文件内部的结构** 

![image-20220404214453931](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220404214454.png)

偏移量：当前指令从偏移量开始，如截图 invokespecial 就占了三个位置（指令占一个位置，指令需要两个位置的参数），return从4开始

操作数： #1 表示常量池中1表示的常量（在javap 命令后面加上 -verbose 才会显示常量池常量）

load指令：从局部变量表加载到栈上, aload_0：把局部变量表0号位置的变量加载到栈上

store指令：从栈运算完之后存回到局部变量表

一个方法对应一个栈帧

![image-20220404211821116](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220404211821.png)

字节码的运行时结构：JVM是一台基于栈的计算机器，每个线程都有一个独属于自己的线程栈（JVM Stack）,用于存储栈帧（Frame）。

>  每一次方法调用、JVM都会自动创建一个栈帧。
>
> 栈帧由操作数栈、局部变量数组以及一个 Class 引用组成。
>
> >  Class 引用指向当前方法在运行时常量池中对应的 Class。

### 类加载器

![image-20220404224553259](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220404224553.png)

类的生命周期的7个步骤，前五个为加载过程的5个步骤

> 加载 -> 验证 -> 准备 -> 解析 -> 初始化 -> 使用 -> 卸载

**三类加载器：**

1. 启动类加载器（BootstrapClassLoader）
2. 扩展类加载器（ExtClassLoader）
3. 应用类加载器（AppClassLoader）

**加载器特点：**

1. 双亲委派机制
2. 负责依赖
3. 缓存加载

### JVM 内存模型

**哪些在栈上存储？哪些在堆上存储？**

栈：原生数据类型、对象引用地址。堆：对象、对象成员与类定义、静态变量。

**栈帧里面有什么？**

操作数栈、局部变量数组以及Class 引用组成的。

**堆内存分为哪两部分？**

年轻代、老年代

**年轻代分为哪三部分？**

Eden区、存活区S0、存活区S1

**非堆是堆吗？是否归GC管理？里面包含哪三个内存池？**

是堆。不归GC管理。元数据区metaspace（以前叫永久代），css，code cache。

**元数据区里面有什么？方法区里面有什么？**

方法区；常量池（Class文件常量池、运行时常量池、基本类型包装类对象常量池）

#### **什么是 JMM ？**

**JMM: Java 内存模型和线程规范，是Java规范的一种；用来控制我们线程之间的交互操作，保障了线程之间的原子性、可见性、有序性，提供了关键字 volatile、synchronized，让大家能够有效的相互之间做协调调度，最后产生符合预期的结果。**

在 JVM 的内部对整个内存进行各种不同的划分，来存放各种不同类型的数据，最后使用 JMM 的规定对大家能够共享和在线程之间传递的这些变量进行统一的规范和管理。一方面能够让我们更高效的使用 JVM 的内存，另外一方面在多核环境下，能够让我们的系统在多核多 CPU 的环境下，让我们的程序能够更高效，同时并发的各种操作产生的结果是可以预期的。不管在不同的操作系统上，不同核数的机器上，还是在不同厂商提供的JVM实现上，同样的代码，产生的结果最终都是一致的。



### JVM启动参数

- -Xss  设置线程栈的最大内存

- -Xmx 设置堆内存的最大值

  > XMX最好配置为最大内存的60%-80%，如果超过80%,有可能会导致OOM内存溢出，因为JVM除了使用堆内存，还有非堆内存，直接内存和自身占用需要的一些内存。

- -Xms 设置堆内存的初始值；

  > 建议配置和 Xmx 一样大，有效防止内存扩容导致的 GC 抖动（大量的对象创建又马上被释放，引起多次 GC，内存扩容），有效预防内存溢出；

- -Xmn 设置堆内存中的年轻代的最大值

  > 官方建议设置为 Xmx 的 1/2 到 1/4 ；JVM默认 Xmn参数是1/3: 即年轻代和老年代的比例是 1:2。

- -XX: MetaSpaceSize=size 设置的是元数据区

  > Java8 默认不限制 Meta 空间，一般不允许设置该项。

- -XX: MaxDirectMemorySize=size 设置JVM的的最大堆外内存

  > 非堆内存不受GC管理；
  >
  > 默认情况下最大可使用到配置的最大堆内存同样大的内存量；实际情况下应用程序不会使用到这么多的直接内存。和 -Dsun.nio.MaxDirectMemorySize 配置效果相同。

选择GC启动：

![image-20220405151633840](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220405151633.png)

**想要出现问题时，自动dump内存快照的配置**

![image-20220405152233281](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220405152233.png)



### JVM 命令行工具

![image-20220405193215896](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220405193215.png)

先用 **jps** 查看进程pid，**jstat** 可以根据pid 按时间间隔循环输出查看gc和堆空间使用信息，以查看堆内存的变化， **jmap** 相当于打了一个快照，可以看到堆空间的分配信息，和类占用大小。**jstack** 根据pid 查看当前java进程中所有线程和锁信息, **jcmd** 可以对jvm的具体状态进行分析，查看具体参数信息。

> **jps -lmv** 查看出所有java进程和对应的 pid，再根据 pid 通过 jstat 查看内部 gc 相关信息
>
> **jstat -gc 和 jstat -gcutil** 没太大区分，都是展示堆内存信息，-gc是展示字节数，单位是KB，-gcutil展示的是百分比。
>
> > 如：jstat -gcutil 18664 1000 10  打印 pid 为 18664 的java进程的堆内存信息，间隔1000毫秒，打10次
> >
> > ![image-20220405195209890](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220405195209.png)
> >
> > ![image-20220405195020302](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220405195020.png)
> >
> > -gc 选项显示 区别不大
> >
> > ![image-20220405200000485](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220405200000.png)
>
> **jmap** 查看堆内存空间信息
>
> ![image-20220405200455561](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220405200455.png)
>
> **jstack** -l pid 查看当前java进程的所有线程信息。
>
> ![image-20220405202548303](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220405202548.png)
>
> **jcmd** pid help，可以看当前可以执行查看的所有命令，执行对应的命令，如 jcmd  pid VM.version 查看对应java进程的jvm版本信息。
>
> ![img](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220405203359.png)
>
> 

### JVM 图形化工具

#### JDK 内置图形化工具

jvisualvm比jconsole多了抽样器功能，**jmc需要重点掌握**，飞行记录器可以记录一段时间内 CPU、内存、代码、线程、IO等具体的变化，如发生了多少次GC，GC暂停时间等等。

jvisualvm和jmc在2020年7月之后发布的jdk版本是没有默认自带的，需要手动下载。

**jconsole**

> 可以看到堆、CPU、线程等概要和具体信息，堆中还可以看到Eden区和存活区等具体的内存分配使用信息。
>
> ![image-20220406140512819](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220406140512.png)

**jvisualvm** （可以使用idea插件VisualGC代替，功能类似）

> 类似jconsole，但有抽样器功能，可以对一段时间内的CPU、内存等情况抽样，还可以Dump堆信息查看。
>
> ![image-20220406142223740](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220406142223.png)

**jmc**

> 类似jconsole，但有飞行记录器功能，可以查看到一段时间内 CPU、内存、代码、线程、IO等具体的变化。
>
> ![image-20220406143307689](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220406143307.png)
>
> 飞行记录器
>
> 可以查看到如发生了多少次GC，GC暂停时间等等，比较强大。
>
> ![image-20220406143630482](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220406143630.png)



### JVM GC

#### GC 的一般原理

引用计数

> 将资源被引用的次数记录，当没有人引用了计数为0，就可以将其释放回收，但存在资源相互引用导致计数永远大于0无法被回收的问题（内存泄露）。

引用跟踪

> 解决引用计数相互引用的问题，从根对象出发，所有可达的对象，就是正在被使用的对象，其它的对象资源则可以被回收。**引用跟踪的实现一般叫：标记清除算法**

**标记清除算法 （并行GC和CMS GC的基本原理）**

> 引用跟踪的算法实现，分为两步
>
> 1. Marking 标记： 遍历所有的可达对象，在本地内存中分门别类记下。
> 2. Sweeping 清除：这一步保障了不可达对象占用的内存，之后可以重新进行分配使用。
> 3. 一般还要做压缩（碎片整理），使得内存布局更紧凑，方便后面申请一大片连续的内存空间。
>
> 如何保障标记和清除 JVM 中上百万的对象？答案就是 STW ，stop the world。
>
> ![image-20220406151201062](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220406151201.png)

根据对象的特性，JVM将内存划分的多个区域，如下图

![](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220406152218.png)



**GC时对象内存池转移概述：**

- 当新生代 Eden 区快满的时候，需要进行**Young GC**：

  先进行 **标记**，将存活的对象从 Eden区 **复制** 到其中一个存活区S0，**清理** Eden 区。

  PS: 为什么是复制，因为复制后原有的数据可以直接清空掉，不需要移动。

- 当Eden区再次满的时候，再次对 Eden 和 S0 区进行标记，将 Eden 区和S0中存活的对象复制到 S1，清理 Eden 和 S0 区。

- 当 Eden 区再满的时候，同上，这次对 Eden 区和 S1 区标记，复制到 S0 区。故 **存活区 S0 和 S1 同一时间必有一个是空的，一般 Eden:S0:S1=8:1:1**。

- 当对象存活超过一定的标记清理周期（默认15次），则会被复制到老年代中而不是另一个存活区。

- 老年代默认都是存活对象，当老年代满了则发生 **Old GC**：

  先**标记**所有通过 GC roots 可达的对象，**清理**所有不可达对象，整理老年代的框架，将所有存活的对象**移动**到老年代空间开始的地方，即做了碎片整理（内存压缩）。PS: 为什么是移动，因为老年代没有分区，通过移动是比较合适的做法。

![image-20220406152309600](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220406152309.png)

可以作为 GC ROOTS 的对象：

![image-20220406160541809](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220406160541.png)

#### 各种不同的 GC 收集器

**串行 GC**

> JVM 启动参数 -XX:+UseSerialGC
>
> 串行GC对年轻代使用 mark-copy (标记-复制)算法，对老年代使用 mark-sweep-comact (标记-清除-整理) 算法。
>
> 由于是单线程，效率不高，后面对串行 GC 做了**并行**改进，在此基础上，对**年轻代使用了多线程**进行 GC 处理，提高了效率。启动参数 -XX:+UseParNewGC 可以配合 CMS GC 使用。

**并行 GC** （JDK8 默认GC）

> JVM 启动参数 -XX:+UseParallelGC
>
> 在串行 GC 的基础上，使用了多线程 GC 进行处理，年轻代和老年代默认都是多线程的，线程数默认为 CPU 核心数，也可使用 -XX:ParallelGCThreads=N 指定线程数。

**CMS GC**（最大可能性的并发的标记清除垃圾回收算法）

**吞吐量较于并行 GC 更小，但是单次 STW 时间更短。**高级的 GC 算法更多追求的是单次 STW 更小。

CMS GC 适合在多核 CPU 且追求更短的 STW 时间，这个时候可以用于代替 并行 GC。

> JVM 启动参数 -XX:+UseConcMarkSweepGC
>
> 对年轻代使用的是 **并行 STW 方式**（即并行改进的串行算法UseParNewGC）的 make-copy（标记-复制）算法，对老年代主要使用 **并发** make-sweep（标记-清除）算法
>
> 相对于 并行 GC：
>
> 1.老年代不做碎片整理操作，而是使用了一个 free-lists 的索引列表记录了当前所用空间的位置，这样少了内存碎片整理这一步，减少了老年代的暂停时间。
>
> 2.在老年代标记清除阶段的大部分工作是和应用线程一起并发执行的。并发线程数是 CPU 核心数的1/4。
>
> ![image-20220406180800768](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220406180800.png)
>
> **CMS GC 的六个阶段**
>
> 1. 初始标记（**伴随STW**）
>
>    > 标记三类对象：根对象、根对象直接引用的对象和年轻代指向老年代的对象。
>    >
>    > 年轻代指向老年代的对象由 JVM 内部的 Remember Set 记录。
>
> 2. 并发标记
>
>    > 接着找引用对象，但此时是并发的，对象的引用关系都在变化，最终标记的状态不一定是非常精确的。
>
> 3. 并发预清理
>
>    > 识别出发生变化的区域，标记为脏区，即 卡片标记。同样也是并发的
>
> 4. 最终标记（**伴随STW**）
>
>    > 把脏区的引用关系完全弄清除。
>    >
>    > 通常 CMS 会尝试在年轻代尽可能空的情况下执行 Final Remark 阶段，以免连续触发多次 STW 事件。
>
> 5. 并发清除
>
>    > JVM 在此阶段删除不再使用的对象，并回收他们占用的内存空间。
>
> 6. 并发重置
>
>    > 重置 CMS 算法相关的内部数据和状态，为下一次 GC 循环做准备。
>
> ![image-20220406205002391](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220406205002.png)
>
> **总结：CMS 垃圾收集器在减少停顿时间上做了很多复杂而有用的工作，用于垃圾回收的并发线程执行的同时，并不需要暂停应用线程。当然，CMS 也有一些缺点，最大的问题是老年代使用的是 标记-清除 算法，因为不整理，会由于内存碎片导致在某些情况下，GC 会造成不可预测的暂停时间，特别是堆内存较大的情况下，申请比较大的内存，有可能导致 GC 时间不可控制。**

**G1 GC**

> 全称：Garbage-First G1的1指的是First，意为垃圾优先，哪一块的垃圾最多就优先清理它。
>
> 可以设置预期 GC 暂停的时间 -XX:+UseG1GC -XX:MaxGCPauseMillis=50 默认200ms
>
> 每个内存块都可能会被定义为Eden、存活区或者Old区；**每次 GC 暂停都会收集所有年轻代的内存块，一般只包含部分老年代的内存块**
>
> **创新就是在并发阶段会估算每个内存块的垃圾数量，优先收集垃圾数最多的内存块。**
>
> 默认划分为2048块，也可以指定每一块的大小（默认堆内存的1/2000）
>
> -XX:ConcGCThreads: 与应用一起执行的GC线程数，默认为CPU核数1/4
>
> ![image-20220407105757983](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220407105758.png)
>
> ![image-20220407105959558](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220407105959.png)
>
> ![image-20220407110438174](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220407110438.png)
>
> ![image-20220407112036220](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220407112036.png)
>
> ![image-20220409143736073](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220409143744.png)
>
> ![image-20220407112521524](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220407112521.png)

**ZGC/Shenandoah GC**

> ZGC 在java 11 linux 版才有，windows和mac 在java15才有
>
> ![image-20220407142258321](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220407142258.png)
>
> ![image-20220407143557985](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220407143558.png)

#### 各 GC 对比

如何选择：在测试系统上做压测，测试不同的组合，看哪种组合最好；没有测试系统则根据吞吐量优先或者低延时优先，再根据应用系统的配置进行对应的选择。

![image-20220407143755511](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220407143755.png)

![image-20220407144004526](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220407144004.png)

![image-20220407190547748](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220407190547.png)



#### GC日志分析

JVM 调优分析最重要的三点：GC 日志看 GC 情况怎么样；线程栈、线程运行的情况怎么样；内存的使用、情况怎么样。

> 第一次GC的日志如下：
>
> >  0.487: [GC (Allocation Failure) [PSYoungGen: 262144K->43518K(305664K)] 262144K->71891K(1005056K), 0.0171261 secs] [Times: user=0.05 sys=0.05, real=0.02 secs] 
> >
> > 在 0.487 这个时刻分配内存失败导致GC，GC之后 |年轻代 262144K -> 43518K（最大容量305664K）|整个堆内存占用262144K -> 71891K（最大容量1005056K） GC暂停时间17ms
> >
> > 由此可计算出老年代的数据，如老年代的容量 = 1005056K - 305664K；第一次晋升老年代的数据= 年轻代被清理的大小 - 堆被清理的大小 即 （ 262144K - 43518K）- （262144K - 71891K）
>
> > 为什么第一次GC 就存在晋升老年代的对象？
> >
> > 对象优先在Eden分配，且新生代对象晋升到老年代有多种情况：
> >
> > (1)、Eden区满时，进行Minor GC，当Eden和一个Survivor区中依然存活的对象无法放入到Survivor中，则通过分配担保机制提前转移到老年代中。 担保机制：就是Minor GC时，新的对象放不下存活区的时候，把存活区的对象晋升为老年代，然后把新对象放到存活区。
> >
> > (2)、若对象体积太大, 新生代无法容纳这个对象，-XX:PretenureSizeThreshold即对象的大小大于此值, 就会绕过新生代, 直接在老年代分配, 此参数只对Serial及ParNew两款收集器有效。
> >
> > (3)、长期存活的对象将进入老年代。
> >
> > ​    虚拟机对每个对象定义了一个对象年龄（Age）计数器。当年龄增加到一定的临界值时，就会晋升到老年代中，该临界值由参数：-XX:MaxTenuringThreshold来设置。
> >
> > ​    如果对象在Eden出生并在第一次发生MinorGC时仍然存活，并且能够被Survivor中所容纳的话，则该对象会被移动到Survivor中，并且设Age=1；以后每经历一次Minor GC，该对象还存活的话Age=Age+1。
> >
> > (4)、动态对象年龄判定。
> >
> > ​    虚拟机并不总是要求对象的年龄必须达到MaxTenuringThreshold才能晋升到老年代，如果在Survivor区中相同年龄（设年龄为age）的对象的所有大小之和超过Survivor空间的一半，年龄大于或等于该年龄（age）的对象就可以直接进入老年代，无需等到MaxTenuringThreshold中要求的年龄。
>
> Full GC 情况：
>
> > 1.074: [Full GC (Ergonomics) [PSYoungGen: 38633K->0K(232960K)] [ParOldGen: 607153K->343126K(699392K)] 645786K->343126K(932352K), [Metaspace: 3396K->3396K(1056768K)], 0.0735577 secs] [Times: user=0.44 sys=0.00, real=0.07 secs] 
> >
> > 类似，新生代直接清空了，还显示了老年代GC的细节和元数据空间的情况。



### JVM 分析调优经验

#### 分配速率

![image-20220409170824251](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220409170824.png)

#### 提升速率

![image-20220409171706794](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220409171706.png)

![image-20220409171853452](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220409171853.png)

![image-20220409172953503](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220409172953.png)

#### JVM 诊断分析工具 - 阿里 Arthas （阿尔萨斯） 

**排查问题的步骤：**

![image-20220409202857416](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220409202857.png)



### 面试题

#### CMSGC和串行 并行的关系,G1GC、ZGC 各种GC适用的场景，java8 默认的GC是什么，G1是哪个里面的

> - **JDK 1.8 默认是ParallelGC ，年轻代 Parallel Scavenge 老年代Parallel old ，JDK 1.9 开始默认采用的就是 G1 垃圾回收器了。**
> - G1 GC 是升级版的 CMS GC ，是增量式的垃圾优先的收集器，有更低的停顿时间，可以指定期望的停顿时间，但不能保证。
> - ZGC 的停顿时间可以保证在 100ms 之内。 收集器的选择一般都是根据业务需求来的。



#### 什么是内存泄漏？什么是内存溢出

**内存泄漏（Memory Leak）**是指本来无用的对象却继续占用内存，没有再恰当的时机 释放占用的内存。

**内存溢出（OOM）**是指可用内存不足。

**如果存在严重的内存泄漏问题，随着时间的推移，则必然会引起内存溢出**。



#### 内存溢出（OOM） 了怎么办？

先恢复系统服务，查看机器内存、CPU、磁盘IO、和网络等情况，查看gc日志信息，监控信息。线上可部署阿里的athers服务，再观察相关堆栈内存信息，及有没有死锁，存活对象排序情况。分析确认是否代码问题，是否可调整JVM参数优化或业务代码优化。不能改造代码的，需要分析数据量情况，考虑扩充机器内存，及重新分配相应内存空间。

增加两个参数 -XX:+HeapDumpOnOutOfMemoryError - XX:HeapDumpPath=/tmp/heapdump.hprof，当 OOM 发生时自动 dump 堆内存信息 到指定目录。 

 同时 jstat 查看监控 JVM 的内存和 GC 情况，先观察问题大概出在什么区域。 

 使用 MAT 工具载入到 dump 文件，分析大对象的占用情况，比如 HashMap 做缓存未 清理，时间长了就会内存溢出，可以把改为弱引用。



#### 频繁 Full GC 怎么操作

> 频繁 Full GC 的原因就是老年代很快就满了，内存不够触发GC，可以根据GC日志进行排查
>
> 1. GC 后剩余的空间很大，说明大部分对象都可以被回收，可以查看下新生代的配置大小，考虑增大新生代的内存，避免频繁 Young GC 导致晋升老年代；也有可能是业务加载过多大对象，考虑增加服务器内存。
> 2. GC 后剩余空间很小，说明有部分对象一直没法被回收，导致可用内存越来越小，可以配合 dump 文件排查内存泄露的问题。







## Java 并发和多线程

#### 线程的状态

![image-20220413134226082](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413134226.png)

![image-20220413134244848](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413134244.png)



join 内部其实调用了 wait 方法，都会将当前的锁释放

#### **volatile 关键字**

1. 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

   synchronized 和 Lock 也能保证可见性，因为同一时刻，只有一个线程获取锁然后执行同步代码，释放锁之前会将变量的修改刷新到主存当中。

   > 两个线程，线程1读取了变量stop的值，会将stop变量的值拷贝一份放在自己的工作内存当中；当线程2更改了主内存stop变量的值之后，但是还没来得及写入主存当中，线程2转去做其他事情了，那么线程1读取的依旧还是原来的值，读取不到线程2修改的值。
   >
   > 　　但是用volatile修饰之后就变得不一样了：
   >
   > 　　第一：使用volatile关键字会强制将修改的值立即写入主存，那么线程1读取的时候立刻就能知道；
   >
   > 　　第二：使用volatile关键字的话，当线程2进行修改时，会导致线程1的工作内存中缓存变量stop的缓存行无效（反映到硬件层的话，就是CPU的L1或者L2缓存中对应的缓存行无效）；
   >
   > 　　第三：由于线程1的工作内存中缓存变量stop的缓存行无效，所以线程1再次读取变量stop的值时会去主存读取。

2. 禁止进行指令重排序。

**volatile保证原子性吗？**

不能保障原子性；能不用就不用，因为修改要里面写入主存，没法利用本地内存的副本，效率不高，可以用juc原子类的相关实现代替。

> 场景：
>
> > 根据缓存一致性，一个处理器的缓存回写到主存会导致其他处理器的缓存失效。当线程A，B同时执行volatile int i的自增的时候，先执行完的线程A回写数据到主存，导致B的缓存变量无效（因此执行+1操作已经没有意义），直接从主存读取最新的值，这样就相当于少做了一次+1。
>
> > 下面这段话摘自《深入理解Java虚拟机》：
> >
> > 　　“观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令”
> >
> > 　　lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：
> >
> > 　　1）它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
> >
> > 　　2）它会强制将对缓存的修改操作立即写入主存；
> >
> > 　　3）如果是写操作，它会导致其他CPU中对应的缓存行无效。

##### volatile可见性实现原理
JMM内存交互层面：

> volatile 修饰的变量的 read、load、use 操作和 assign、store、write 必须是连
> 续的，即修改后必须立即同步回主内存，使用时必须从主内存刷新，由此保证volatile变量的
> 可见性。

底层实现：

>  通过汇编lock前级指令，它会锁定变量缓存行区域并写回主内存，这个操作称为 “缓存锁定”
>
> 1. 缓存一致性机制会阻止同时修改被两个以上处理器缓存的内存区域数据(MESI协议）
> 2. 一个处理器的缓存回写到内存内存会导致其他处理器的缓存无效(MESI协议）

#### volatile 带来的伪共享问题

```java
class Pointer {
    volatile long x;
    volatile long y;
}
```

如果两个线程，分别对 x、y 进行++一万次，由于 volatile 修饰的变量一旦修改会立刻让其它 CPU 缓存失效，从而读取主存的数据，一个缓存行是64字节，那么 x 和 y可能在一个缓存中

线程1 x进行修改后，会让线程 2 对应 x所在的缓存行失效，那么线程2 对 y 的修改也在这个缓冲中，本次修改就无效了，x 和 y 并没有关系，但是确会相互影响

**解决方法**

让两个数 不在一个缓存行中

- 中间再放7个 long变量，这样 x 和 y 怎么都不会在一个缓存行中。

- 封装类，计算好类大小，保证不在一个缓存行

  ```java
  
  class Pointer {
      MyLong x = new MyLong();
      MyLong y = new MyLong();
  }
   
  class MyLong {
      volatile long value;
      long p1, p2, p3, p4, p5, p6, p7;
  }
  ```

- 开启注解

  ```java
  //默认使用这个注解是无效的，需要在JVM启动参数加上-XX:-RestrictContended才会生效
  @sun.misc.Contended
  class MyLong {
      volatile long value;
  }
  ```

前两种方法可能会被 jvm 优化导致失效。



#### synchronized 关键字

synchronzied 也称为同步锁，可以加在变量、方法或一块代码上，被修饰的对象资源同一时刻只会有一个线程执行，达到并发安全的效果。

synchronized保证了并发的原子性、可见性和有序性：

- 原子性：确保线程互斥地访问同步代码；
- 可见性：保证共享变量的修改能够及时可见，其实是通过Java内存模型中的“对一个变量unlock操作之前，必须要同步到主内存中；如果对一个变量进行lock操作，则将会清空工作内存中此变量的值，在执行引擎使用此变量前，需要重新从主内存中load操作或assign操作初始化变量值” 来保证的；
- 有序性：有效解决重排序问题，即 “一个unlock操作先行发生(happen-before)于后面对同一个锁的lock操作”；

资源的对象头的标记位中，里面记录了锁状态等信息。

轻量级锁、重量级锁

> https://blog.csdn.net/weixin_29240343/article/details/112579058
>
> https://blog.csdn.net/qq_38826019/article/details/119307386
>
> https://blog.csdn.net/m0_56651882/article/details/119717738



![image-20220413160328396](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413160328.png)

##### 偏向锁的原理是什么?

>  当锁对象第一次被线程获取的时候，虚拟机将会把对象头中的标志位设为“01”，即偏向模式。
>
> 同时使用CAS操作把获取到这个锁的线程的ID记录在对象的Mark Word之中 ，
>
> 如果CAS操作成功，持有偏向锁的线程以后每次进入这个锁相关的同步块时，虚拟机都可以不再进行任何同步操作，偏向锁的效率高。

##### 偏向锁的好处是什么?
>  偏向锁是在只有一个线程执行同步块时进一步提高性能，适用于一个线程反复获得同一锁的情况。
>
> 偏向锁可以提高带有同步但无竞争的程序性能。

##### 偏向锁获取过程-升级轻量锁

1. 访问Mark Word中偏向锁的标识是否设置成1，锁标志位是否为01，确认为可偏向状态。
2. 如果为可偏向状态，则测试线程ID是否指向当前线程，如果是，进入步骤5，否则进入步骤3。
3. 如果线程ID并未指向当前线程，则通过CAS操作竞争锁。如果竞争成功，则将Mark Word中线程ID设置为当前线程ID，然后执行5；如果竞争失败，执行4。
4. 如果CAS获取偏向锁失败，则表示有竞争。当到达全局安全点（safepoint）时获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。（撤销偏向锁的时候会导致stop the world）
5. 执行同步代码。

**注意**：第四步中到达安全点safepoint会导致stop the world，时间很短。

##### 什么是轻量级锁

> 轻量级锁是JDK 6之中加入的新型锁机制，它名字中的“轻 量级”是相对于使用monitor的传统锁而言的，因此传统的锁机制就称为“重量级”锁。
>
> 首先需要强调一点的是，轻量级锁并不是用来代替重量级锁的。
>
> 引入轻量级锁的目的：在多线程交替执行同步块的情况下，尽量避免重量级锁引起的性能消耗，但是如果多个线程在同一时刻进入临界区，会导致轻量级锁膨胀升级重量级锁，
>
> 所以轻量级锁的出现并非是要替代重量级锁

##### 轻量级锁原理

> 当**关闭偏向锁功能或者多个线程竞争偏向锁导致偏向锁升级为轻量级锁，则会尝试获取轻量级锁**，其步骤如下： 获取锁
>
> 1. 判断当前对象是否处于无锁状态（hashcode、0、01），如果是，则JVM首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，
>
> 用于存储锁对象目前的Mark Word的拷贝（官方把这份拷贝加了一个Displaced前缀，即Displaced Mark Word），
>
> 将对象的Mark Word复制到栈帧中的Lock Record中，将Lock Reocrd中的owner指向当前对象。
>
> 2. JVM利用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，如果成功表示竞争到锁，则将锁标志位变成00，执行同步操作。
>
> 3. 如果失败则判断当前对象的Mark Word是否指向当前线程的栈帧，如果是则表示当前线程已经持有当前对象的锁，则直接执行同步代码块；
>
> 否则只能说明该**锁对象已经被其他线程抢占了，这时轻量级锁需要膨胀为重量级锁，锁标志位变成10，后面等待的线程将会进入阻塞状态。**

##### 轻量级锁的好处

> 在多线程交替执行同步块的情况下，可以避免重量级锁引起的性能消耗。

##### 轻量级锁的释放
> 轻量级锁的释放也是通过CAS操作来进行的，主要步骤如下：
>
> 1. 取出在获取轻量级锁保存在Displaced Mark Word中的数据。
>
> 2. 用CAS操作将取出的数据替换当前对象的Mark Word中，如果成功，则说明释放锁成功。
>
> 3. 如果CAS操作替换失败，说明有其他线程尝试获取该锁，则需要将轻量级锁需要膨胀升级为重量级锁。
>
> 对于**轻量级锁，其性能提升的依据是“对于绝大部分的锁，在整个生命周期内都是不会存在竞争的**”，如果打破这个依据则除了互斥的开销外，
>
> 还有额外的CAS操作，因此在有多线程竞争的情况下，轻量级锁比重量级锁更慢。

##### 轻量级锁 总结

> **轻量级锁的原理是什么？**
>
> 将对象的Mark Word复制到栈帧中的Lock Recod中。Mark Word更新为指向Lock Record的指针。
>
> **轻量级锁好处是什么？**
>
> 在多线程交替执行同步块的情况下，可以避免重量级锁引起的性能消耗。

##### 锁粗化

> 原则上，我们在编写代码的时候，总是推荐将 同步块的作用范围限制得尽量小，只在共享数据的实际作用域中才进行同步，这样是为了使得需要同步的操作数量尽可能变小，如果存在锁竞争，那等待锁的线程也能尽快拿到锁。
>
> 大部分情况下，上面的原则都是正确的，但是如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作是出现在循环体中的，
>
> 那即使没有线程竞争，频繁地进行互斥同步操作也会导致不必要的性能损耗。

![f091ae81a5f9e4df0e970b4c41318733.png](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220415205017.png)



##### 口头阐述锁升级的过程

当一个线程1第一次访问 synchronized 关键字修饰的变量的时候，会将该对象头的标记字的锁标志修改为偏向锁，并将当前线程id 用CAS方式写入标记字，当同一个线程在不同方法多次获取锁的时候，判断偏向锁的线程id是否为自己，这样获取到锁的时候下一次进来就不需要进行同步操作，直接执行同步块，提高了效率。

当另一个线程2 尝试获取偏向锁的时候，发现线程id不是他，且这个线程id处于存活状态，那么jvm将会在全局安全点的时候，阻塞线程1，将锁升级为轻量级锁，会将线程1的栈帧开辟一个锁记录的内存空间，并复制存放锁对象的标记字，并使用owner指向锁对象，同时标记字将写入指针指向这个锁记录。线程2 在也会复制对象头操作，发现当前锁被占用，尝试CAS自旋重试的方式去获取锁。

CAS自旋也是需要消耗资源的，当自旋次数过多，同时多个线程反复竞争的时候，轻量级锁就会粗化为重量级锁，把其他线程都阻塞，避免自旋空转浪费资源。



#### 线程池

**线程的资源对于系统来说是比较重量级的资源，线程占用资源比较多，所以更倾向于使用线程池重复利用。**

JDK中线程池的抽象定义和具体实现：

![image-20220413164653569](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413164653.png)

1. Excutor

   > 顶层接口，就一个方法 运行一个线程

2. ExcutorService

   > 基于Excutor 扩展的API接口类

3. ThreadFactory

   > ThreadPoolExecutor中用工厂模式创建线程

4. ThreadPoolExecutor

   > ExecutorService 实现类

5. Excutors

   > 工具类，供开发使用



**ThreadPoolExecutor的execute 方法如下：**

提交任务分为三步：

- 首先判断线程池运行的线程数**是否小于核心线程数**，小于则尝试创建一个核心线程执行任务。
- 如果超过核心线程数，则**加入工作队列**，加入成功时会判断当前线程池是否处于运行状态，若非运行状态则丢弃任务并执行拒绝策略。
- 工作队列已满，则会再次尝试创建非核心线程，将判断**是否超过最大线程数**，超过最大线程数则**执行拒绝策略**

超过核心线程数创建的线程将指定 allowCoreThreadTimeout=true ，空闲时间超过空闲存活时间则会被回收。

![image-20220413172334429](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413172334.png)

![](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220414234018.png)

创建线程的时候，会根据创建的是核心线程还是非核心线程比较对应的容量

![image-20220415085254518](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220415085254.png)

![image-20220414233545944](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220414233546.png)

拒绝策略主要有以下四种：

AbortPolicy 是默认的拒绝策略

![image-20220413172308075](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413172308.png)

![image-20220413180453233](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413180453.png)

![image-20220413180648671](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413180648.png)

仅仅 newCachedThreadPool 有指定默认60秒的空闲存活时间。

![image-20220413205712385](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413205712.png)

![image-20220413205850303](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413205850.png)

#### juc 包

![image-20220413213622617](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413213622.png)

五大类对应的实现类

![image-20220413214124495](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413214124.png)

##### 锁

什么是锁?

> 锁是保障多线程下对资源并发操作的前提，能够保障并发的性质：原子性、可见性和有序性。

可重入锁：线程获取拿到了锁，没有释放锁，经过某次调用又调用到了这部分代码块，线程不会被阻塞。

公平锁：按照排队靠前的优先拿到锁。

ReentrantLock：可重入锁。

ReentrantReadWriteLock：读写锁

> readLock() 读锁，共享锁
>
> writeLock() 写锁，排他锁

Condition：作用在 Lock 上面的 wait/notify 机制。它比 Lock.lock() 这种方法更加灵活 可以利用一个Lock根据不同条件分开wait/signal,而不需要多个 Lock 去加锁。

LockSupport: 类似工具类，提供类各种锁方法。

> - 如果 synchronized 关键字适合你的程序， 那么请尽量使用它，这样可以减少编写代码的数量，减少出错的概率。因为一旦忘记在 finally 里 unlock，代码可能会出很大的问题，而使用 synchronized 更安全。
> - 如果特别需要 Lock 的特殊功能，比如尝试获取锁、可中断、超时功能等，才使用 Lock。
> - synchronized 不需要手动上锁和释放锁，Lock 更加灵活。

![image-20220416142817749](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220416142817.png)



![image-20220413223026066](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220413223033.png)

##### CAS

CAS 是一种无锁技术

- CAS compare and swap 表示的是 比较并交换
- CAS 操作包括三个数 内存地址、期望原值和新值。如果内存位置的值和期望原值相等则写入新值。
- CAS 是CPU 的硬件指令

自旋锁就是基于 CAS 的一种乐观锁，在 Java 中 Unsafe 是 CAS 的实现类 API，是CAS 一般搭配 volatile 一起使用，在 Java 的并发原子类实现，其核心就是 CAS 加 volatile:

> volatile 保障多线程变量读写的可见性。
>
> 使用 CAS 指令，作为乐观锁实现，通过**自旋重试保证写入**。

CAS 存在 ABA 问题，一般使用版本号（时间戳）解决，**AtomicReference**可以解决 ABA 问题。



##### 并发原子类 Atomic

无锁技术的底层实现原理：

> 1. Unsafe API - CompareAndSwap
> 2. CPU 的硬件指令 - CAS 
> 3. volatile 的可见性 - volatile 关键字
>
> 使用 CAS 指令，作为乐观锁实现，通过自旋重试保证写入。

![image-20220414132450656](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220414132450.png)

##### 并发工具类

并发工具类主要提供以下场景多个线程相互协调相互协作的解决方案，这是单单用锁或者并发原子类不好操作或实现的的：

> 1. 允许并发，但限制并发线程数，如同时只能有3个线程进行。 具体实现为：Semaphore - 信号量
>
> 2. 多个并发的线程在**同一个时间点**一起开始执行；指定多个并发线程**都达到某个状态**再继续处理。
>
>    实现有：CyclicBarrier（非AQS实现）、CountDownLatch

###### AQS（AbstractQueuedSynchronizer）

队列同步器(CLH队列)，是 JUC 并发包中的核心基础组件，**抽象了竞争的资源和线程队列**，通过控制线程队列的行为以及大家共享竞争的资源。

![image-20220414150401568](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220414150401.png)

#### lock 和 synchronized的区别和适用场景

lock 更灵活，可以打断，trylock方法也可以设置超时 lock支持公平和非公平锁

synchronized 不可以打断 不可以获取到锁的状态

synchronized是可重入非公平锁，没有浪费线程唤醒阶段的时间，执行新调用的方法，增加吞吐量，缺点是可能会造成饥饿锁。

#### 线程池的作用？

1. 降低资源消耗：通过重复利用现有的线程来执行任务，避免多次创建和销毁线程。
2. 提高相应速度：因为省去了创建线程这个步骤，所以在拿到任务时，可以立刻开始执行。
3. 提供附加功能：线程池的可拓展性使得我们可以自己加入新的功能，比如说延时执行、周期执行任务。

线程池适用于生存周期较短的的任务，不适用于又长又大的任务。



#### 大量excel文件，文件中一行行的数据，这类数据读取怎么设计

> 首先读取到文件列表，然后针对每一个文件，可以添加任务到线程池中进行处理，每个任务去读取一个文件进行解析，读取文件中的数据进行入库。



## Java 集合

### HashMap

#### HashMap 线程安全问题

读写冲突：两个线程，一个在读一个在写 同一个链表。

扩容问题：扩容后，原先链表的部分数据计算后到新的数组下标中，也会存在读写问题。

#### 是如何保证 HashMap get 方法的时间复杂度是 O(1) 的？

从这个问题展开：

1. hashmap 内部维护有一个 Node 数组，下标是通过key的 hashcode 经过扰动函数得出的hash并 (n - 1) & hash 计算的值，查询数组索引的时间复杂度 O(1) 

   > 扰动函数如下：
   >
   > ```java
   > static final int hash(Object key) {
   >  int h;
   >  return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
   > }
   > ```
   >
   > 数组下标：(n - 1) & hash  n指数组长度

2. 遍历索引对应的键值对链表，根据key的equals方法找到返回值，当链表长度为1，时间复杂度即为O(1)，当链表长度大于8，且索引数组长度大于64的时候，链表自动转换为红黑树，时间复杂度是O(logn)。

3. 如何保证时间复杂度是 O(1)，方向：让元素均匀的铺在 entry 数组中，降低链表的长度，尽量不变成红黑树，那么每次时间复杂度即为 O(1)

   > 如何均匀的分别到数组的各个位置（桶）：
   >
   > 根据扰动函数 (h = key.hashCode()) ^ (h >>> 16)  自身跟自身右移16位的值进行异或
   >
   > 为什么与右移16位进行异或？
   >
   > > hashCode()返回一个int值，占4个字节共32位，右移16位则将只保留了高16位进行计算，前面填充的0 异或不改变结果。
   > >
   > > **目的就是：为了让高16位参与计算，达到让后面计算数组下标  (n - 1) & hash  更加散列的目的**
   > >
   > > ![image-20220411164923816](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220411164930.png)
   > >
   > > 更加详细的解析为什么是高16位：
   > >
   > > > 因为  (n - 1) & hash 计算下标原理类似取余数（前提是n必须是2的幂次），得到数组的位置，比如：
   > > >
   > > > n=8的时候, n-1 等于7 二进制是 0111，hash & (n-1) 如下
   > > >
   > > > ![image-20220411172115226](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220411172115.png)
   > > >
   > > > 由于大部分情况下，length 要小于2的16次方，所以 计算 (n - 1) & hash 往往都是低16位参与计算，所以 HashMap 的扰动函数获取的hash值，让高16位参与了进去，在计算下标的时候能够达到更加散列的目的。

当数据量大，

#### HashMap 的扩容机制是怎样的？ 

一般情况下，当元素数量**超过阈值**时便会触发扩容。**每次扩容的数组容量都是之前容量的 2 倍**。 

HashMap 的容量是有上限的，必须小于 1<<30，即 1073741824。如果容量超出了这个 数，则不再增长，且阈值会被设置为 Integer.MAX_VALUE。 

HashMap 是线程不安全的，HashTable 相当于是 HashMap 的 synchronized 加锁实现。

 **JDK8 的扩容机制** 

- 空参数的构造函数：实例化的 HashMap 默认内部数组是 null，即没有实例化。第一次 调用 put 方法时，则会开始第一次初始化扩容，数组长度为 16。 
- 有参构造函数：用于指定容量。会根据指定的正整数找到不小于指定容量的 2 的幂数， 将这个数设置赋值给阈值（threshold）。第一次调用 put 方法时，会将阈值赋值给容量， 然后让 **阈值 = 容量 x 负载因子**（0.75）。 
- 如果不是第一次扩容，则数组容量变为原来的 2 倍，阈值也变为原来的 2 倍。（容量和阈值 都变为原来的 2 倍时，负载因子还是不变）。 此外还有几个细节需要注意： 
- 首次 put 时，先会触发扩容（算是初始化），然后存入数据，然后判断是否需要扩容；
- 不是首次 put，则不再初始化，直接存入数据，然后判断是否需要扩容；

JDK7 中的扩容机制 

- 空参数的构造函数：以默认容量、默认负载因子、默认阈值初始化数组。内部数组是空数 组。  有参构造函数：根据参数确定容量、负载因子、阈值等。 
- 第一次 put 时会初始化数组，其容量变为不小于指定容量的 2 的幂数，然后根据负载因 子确定阈值。  如果不是第一次扩容，则 新容量=旧容量 x 2 ，新阈值=新容量 x 负载因子 。

### ConcurrentHashMap

#### jdk1.7 分段锁实现

为了提高并发的性能，采用了Segment分段锁，默认为16个Segment，将数据进来先做一层hash，分散到16个段里面，给每一个段加锁，这样可以降低锁的粒度，一个段相当于一个单独的hashmap，线程往这个段写的时候，其它线程可以读取到其他段里面的数据。

#### jdk1.8 实现

利用的是 CAS 自旋的机制，给 tab 加上 volatile 修饰。

当对应数组下标为null，则采用CAS方式将Node放到对应位置，如果下标存在对象则是用synchronized上锁，然后操作链表。

1. 为了兼容保留了 Segment。
2. 底层结构存放的是TreeBin对象，而不是Node对象。
3. ConcurrentHashMap实现中借用了较多的CAS算法，unsafe.compareAndSwapInt(this, valueOffset, expect, update); CAS(Compare And Swap)，意思是如果valueOffset位置包含的值与expect值相同，则更新valueOffset位置的值为update，并返回true，否则不更新，返回false。
4. 在操作[hash](https://so.csdn.net/so/search?q=hash&spm=1001.2101.3001.7020)值相同的链表的头结点还是会synchronized上锁，这样才能保证线程安全。



#### CopyOnWriteArrayList

读写分离，读未上锁；写操作，使用了 ReentrantLock 可重入锁，复制了一个快照，在快照上写，不影响读，写完再将数组引用指向快照。



### ThreadLocal

类似 hashmap，内部定义了一个 ThreadLocalMap ，map 是根据线程的id创建而来的，所以每个线程之间是隔离的。

![image-20220414184736082](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220414184736.png)





## 关系型数据库MySQL

#### MySQL 存储引擎

![image-20220430120452815](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220430120500.png)

#### MySQL 的语句执行过程

![img](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220419214754.png)

#### SQL 执行顺序

![image-20220430120617686](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220430120617.png)



#### MySQL 的ACID 分别由什么保证？

> **原子性（Atomicity）**：事务是一个原子操作 ，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
>
> > **由 undolog 日志来保证，它记录了需要回滚的日志信息，事务回滚时撤销已经执行成功的sql。**
>
> **一致性（Consistency）**：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
>
> > 是由其他三大特性保证，程序代码要保证业务上的一致性。
>
> **隔离性（Isolation）**：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
>
> > 是由 MVCC 来保证。
>
> **持久性（Durability）**：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。
>
> > 由 redolog 来保证，mysql 修改数据的时候会在 redolog 中记录一份日志数据，就算数据没有保存成功，只要日志保存成功了，数据仍然不会丢失。
> > 

#### MySQL 索引 B+ 树结构

InnDB 使用 B+ 树实现聚簇索引，B+ 树是一个稳定有序的平衡二叉树，由叶子节点指向记录，叶子节点以上为索引，叶子节点构成一个有序的双向链表。

数据页的存储是按块进行存储的，一页就是一块，如下聚簇索引，一个数据块中能容纳多个key+指针组合的数据，每一个指针又指向一个新的数据块。

**InnoDB默认定义的块大小是16KB，通过innodb_page_size参数指 定。这里所说的“块”，是一个逻辑单位，而不是指磁盘扇区的物理块。 块是InnoDB读写磁盘的基本单位，InnoDB每一次磁盘I/O，读取的都是 16KB的整数倍的数据。无论叶子节点，还是非叶子节点，都会装在 Page里**

![image-20220420085159285](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220420085159.png)

![image-20220420104518414](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220420104518.png)

**非聚簇索引叶子节点存放的是主键信息，查询还需要再去聚簇索引去查一遍找到数据记录，也就是回表。**



#### 为什么一般单表数据不超过2000万？

一般情况下，MySql三层的索引结构是性能比较好的极限值，再加一层索引结构性能的损耗比较大，最后评估三层的索引结构对于一张表来说数据在2100万左右，所以为了保持较好的性能，单表最好不要超过2000万条数据。

三层：两层的索引+一层的数据块。

> 一个数据块 默认 16k，如果主键是bigint 占8个字节，指针大概占6个字节，则一块最多可放 16*1024/(8+6) = 1170 个索引，两层索引总数 1170 *1170，然后假设一个数据记录占1k，则一个数据块可存储16个记录，则 三层结构的总数据记录为：1170 * 1170 * 16 = 21902400 大约2200万条记录。



#### 大批量导入数据文件

```sql
-- 1.关闭唯一性校验、删除掉索引
SET UNIQUE_CHECKS=0;
-- 2.改为手动提交事务
SET AUTOCOMMIT=0;
-- 3.导入数据
-- CHARACTER SET utf8 -- 可选，避免中文乱码问题
-- FIELDS TERMINATED BY ',' -- 字段分隔符，每个字段(列)以什么字符分隔，默认是 \t
-- OPTIONALLY ENCLOSED BY '' -- 文本限定符，每个字段被什么字符包围，默认是空字符
-- ESCAPED BY '\\' -- 转义符，默认是 \
-- LINES TERMINATED BY '\r\n' -- 记录分隔符，如字段本身也含\n，那么应先去除，否则load data 
load data local infile 'F:\\test\\t_user.csv' into table `t_user` fields terminated by ',' OPTIONALLY ENCLOSED BY '"' lines  terminated by '\r\n';
-- 4.自动提交事务
SET AUTOCOMMIT=1;
-- 5.打开唯一性校验、建立索引
SET UNIQUE_CHECKS=1;
```



#### MySQL 大表新增列

新增列不影响索引，可以在低峰期直接加，MySQL 5.6 后支持 online ddl，如果有从库，可以先下一台从库进行测试。



#### MySQL 锁

##### 表级锁

###### 意向锁

上锁之前需要先上意向锁，表面本次事务稍后要进行哪种类型的操作，这样其它事务过来，先判断意向锁，如果意向锁冲突，则就不需要再去执行获取更细粒度的锁了。

> - 共享意向锁（IS）：打算在某些行上设置共享锁
> - 排他意向锁（IX）：打算在某些行上设置排他锁
> - Insert 意向锁：Insert 操作设置的间隙锁

其他表级锁：

> - 自增锁
> - LOCK TBALES/DDL语句

**判断 锁的冲突和兼容性的依据：**

> - X 锁 都冲突  
>
>   > select for update 就是一个X锁
>
> - IS锁 除了与X锁 都兼容
>
> - S锁 和其他共享锁都兼容 和排他锁都不兼容
>
> - IX锁 只和IX锁和IS锁兼容

![image-20220420095035853](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220420095035.png)

##### 行级锁

https://blog.csdn.net/weixin_37686415/article/details/114711276

Gap 锁可参考：

>  https://blog.csdn.net/u022812849/article/details/122528923
>
> https://www.jianshu.com/p/d1aba64b5c03
>
> - 等值查询：针对唯一索引，如果该记录不存在，会产生间隙锁，如果记录存在，则只会产生记录锁；而普通索引会产生间隙锁
> - 对于查找某一范围内的查询语句，不管什么索引都会产生next-key锁，普通索引可以使用limit减少锁区间范围。
> - 如果列上无索引会导致锁表。

> ![image-20220420095556676](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220420095556.png)



#### MySQL 事务隔离级别

- 读未提交

  > 多个事务，一个事务修改的还未提交，其他事务也看得到。
  >
  > **可能导致脏读、幻读、不可重复读。**

- **读已提交**

  > 多个事务，可以读取到其他事务已提交的数据。
  >
  > **不对select 的数据加锁的情况下：可能导致幻读、不可重复读。**
  >
  > > 不可重复读：不加锁的情况下，其他事务修改了一行记录的数据，导致两次查询结构数据发生变化。
  > >
  > > 幻读：不加锁的情况下，其他事务新增了一批数据，导致两次读出来的记录数量不一致。

- **可重复读 - MySQL 默认级别**

  > 每次读取的数据，更上一次读取的数据是一样的（读取快照的方式）。
  >
  > 不会读取到事务进行中 其他事务提交的数据。
  >
  > 使用MVCC实现
  >
  > ![image-20220420101534716](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220420101534.png)

- 可串行化

  > 不允许事务并发执行，性能比较差。

#### 脏读、幻读、不可重复读

- 脏读

  > 读取到其他事务未提交的数据，随时可能被回滚掉。

- 幻读

  > 第一次读出来3条数据，第二次读出5条数据，每次读出来数据量不一样。

- 不可重复读

  > 两次或多次读出来的数据量是一样的，但每次数据中同一的某一条数据，先后的值不同。

#### 事务的传播行为

1、PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。

2、PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作

3、PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。‘

4、PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。

5、PROPAGATION_REQUIRES_NEW：支持当前事务，创建新事务，无论当前存不存在事务，都创建新事务。

6、PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

7、PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。



#### EXISTS与IN的使用效率对比

in是把外表和内表作hash连接，而exists是对外表作loop循环，每次loop循环再对内表进行查询，一直以来认为exists比in效率高的说法是不准确的。如果查询的两个表大小相当，那么用in和exists差别不大；如果两个表中一个较小一个较大，则子查询表大的用exists，子查询表小的用in；



#### 悲观锁乐观锁

select for update 就是悲观锁，给表上了一个X锁

![image-20220420131757450](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220420131757.png)

#### 如何让1-10序号的数据，小于5逆序 大于5顺序查询出？

```sql
select f_id,f_name 
from t_user 
where f_id < 10
order by if(f_id<5, -f_id, f_id);
```

#### 主键为什么要单调递增？

避免叶子节点的数据块发生页分裂，参考B+树，新插入的数据满了一个数据块就会新增一个数据块，如果不是递增的，则可能插入到一个已经满的数据块中间，导致数据块放不下需要重新分配，进而导致索引结构发生变化，影响性能。

#### 什么是最左原则？

复合索引（A,B)的B+树 都是根据最左边的字段A建立索引结构的，然后在叶子节点上A是有序的双向链表，叶子节点的数据为（A,B,主键)，A相等的情况下 B是有序的。

两个索引 （A, B) 和 （A, B, C） 保留 （A, B, C） 即可。

**有唯一约束的字段 天然就是一个索引。**

**多个字段建立索引，长索引包括短索引，以左对齐进行匹配。**

#### 什么是索引下推？

在组合索引中 (name, phone)，查询 name like 'aa%'  and phone='123'，为减少回表的次数，存储引擎根据组合索引，在遍历索引的时候同时对 phone 做了筛选，过滤掉不符合的记录，再去回表查询记录。



#### 什么是MySQL三大范式?

> 第一范式：
>
> 　　1NF是对属性的原子性，要求属性具有原子性，不可再分解；
>
> 第二范式：
>
> 　　2NF是对记录的唯一性，要求记录有唯一标识，即实体的唯一性，即不存在部分依赖；
>
> 第三范式：
>
> 　　3NF是对字段的冗余性，要求任何字段不能由其他字段派生出来，它要求字段没有冗余，即不存在传递依赖
>
> （1）简单归纳：
>
> 　　第一范式（1NF）：字段不可分；
> 　　第二范式（2NF）：有主键，非主键字段依赖主键；
> 　　第三范式（3NF）：非主键字段不能相互依赖。
>
> （2）解释：
>
> 　　1NF：原子性。 字段不可再分,否则就不是关系数据库;；
> 　　2NF：唯一性 。一个表只说明一个事物；
> 　　3NF：每列都与主键有直接关系，不存在传递依赖。



#### 数据库水平和垂直拆分

垂直拆分：比如将原来一整个库，所有表数据都在一块，进行业务拆分，拆分为三个库：用户库、商品库、订单库。

水平拆分：不影响原来的业务和表设计，只是做表/库的容量扩容。

**垂直拆分改变数据库结构，水平拆分不改变数据结构，仅是讲数据分散开来。**

**分库分表的作用:**

1.容量问题：在数据量特别大的时候，索引层级增加、磁盘IO压力增大，都会导致很多问题。

2.性能问题：索引深度增加，IO次数也随着增加，查询性能下降严重，高并发瓶颈。

3.可用性方面：如果单单靠主从，从运维方面考虑，数据量大导致数据备份和恢复的时间成本不可控，且主从可能延时高。

分库的副作用：将数据分散在不同的数据库节点上，导致很多操作就会分散在不同的数据库上，破坏了一个事务，带来了一致性问题，需要分布式事务来解决该问题。



### 面试题

#### MySQL 如何调优

参考：https://www.cnblogs.com/soft-engineer/articles/15379769.html

- 增大查询缓存大小

- **充分使用索引**

  > 避免查询对索引列进行函数运算、双% like、or、隐式转换等可能导致不走索引的问题；
  >
  > 适当建立索引，对于查询固定两个字段的可以建立组合索引而不需回表。

- **分析查询日志、慢查询**

- 定期重建 mysql 

- 选择合适的存储引擎

#### InnoDB 引擎是否支持 Hash 索引？

- MyISAM和InnoDB 不支持HASH
- InnoDB可以使用`Adaptive Hash`来间接支持HASH
- MEMOEY才能同时支持HASH和BTREE

InnoDB引起有一个特殊的功能叫做"自适应哈希索引（adaptive hash index）" 。当InnoDB注意到某些索引值被使用的非常频繁时，它会在内存中基于B-Tree索引之上再创建一个哈希索引，这样就让B-TREE索引也具有哈希索引的一些优点，比如快速的hash查找，这是一个完全自动的，内部的行为，用户无法控制或者配置，不过，如果有必要，完全可以关闭该功能。

#### 索引为什么不适用hash index的方式建立？

答：**Hash 索引仅仅能满足”=”,”IN”和”<>”查询，不能使用范围查询。没办法保证两个值 hash前大小和hash后是一致的。当hash冲突较多的时候，要定位一条记录的代价并不小。**



#### 事务隔离级别可重复读和读已提交如何选择？

MySQL 默认是可重复读，Oracle 默认是读已提交。

可重复读存在 Gap 锁，增大了锁等待和死锁的概率；读已提交没有 Gap 锁，并发性得到一个好的保证。

如何选择：需要看业务逻辑，比如 修改一组数据 这组数据是否需要保证一致性，如果一致性要求没那么强 可以使用提交读 (事务内 多次查询数据 有可能不一致) ，这样你并发会提高很多 减少了锁冲突的概率。



#### 什么时候行锁会升级为表锁？

参考：https://blog.csdn.net/u022812849/article/details/122528923

行锁是建立在索引字段的基础上

- 等值查询：针对唯一索引，如果该记录不存在，会产生间隙锁，如果记录存在，则只会产生记录锁；而普通索引会产生间隙锁
- 对于查找某一范围内的查询语句，不管什么索引都会产生next-key锁，普通索引可以使用limit减少锁区间范围。
- 如果列上无索引会导致锁表。



#### binlog

binlog指二进制日志，它记录了数据库上的所有改变，并以二进制的形式保存在磁盘中，它可以用来查看数据库的变更历史、**数据库增量备份和恢复、MySQL的复制（主从数据库的复制）**。

binlog记录的是实实在在的sql语句

binlog有三种格式：

statement：基于SQL语句的复制（statement-based replication，SBR）
row：基于行的复制（row-based replication，RBR）
mixed：混合模式复制（mixed-based replication，MBR）

#### redolog

redolog记录的是数据的物理变化，作用主要是用来事务崩溃恢复，将redolog加载到内存里面，内存就能恢复到挂掉之前的数据。

#### undolog

**undo log有什么用？**
undolog 主要用来回滚和mvcc多版本控制

**undolog长啥子样？**
undolog其实存储的也是逻辑日志，比如说我们要insert一条语句，那么undolog就记录着一条delete语句，所以说它可以用来做回滚

同时因为undolog记录的是修改之前的数据，相当于前一个版本，而mvcc实现的是读写不阻塞，读的时候只要返回前一个版本的数据就好了。



#### 为什么B+树更适合做索引？

答：**平衡二叉树本就适合快速查找，且B+树的层级更小，同时叶子节点有双向指针的存在，不需要回溯，降低了开销。**



#### 为什么主键长度不能过大？

答：**主键长度过大，会导致每一个数据块的可存放的key的数量减小，可能会导致B+树的层级变大，极大的影响查询的速度。**



#### 树状结构数据如何设计表

> 1.增加一个 parentid 字段，存放对应父节点的id，根据父节点关系遍历
>
> 2.类似栏目结构，固定三个数字一组，001001，对于001001其父节点就是001，然后兄弟节点就是001% + len=6



#### 聚簇索引的创建逻辑

选取规则：主键索引 ->  非空唯一索引 -> 隐式定义主键建立

聚簇索引就是叶子节点是存放行记录数据的那个索引，叶子节点也被称为数据页。



#### is null、is not null 到底走不走索引

**NULL表示没有设置任何的值，空值其实是有值的，只是值为空而已。**

```mysql
#name字段可为空的情况下 判断走name字段索引
# is null 可能走索引
EXPLAIN select * from student where name is null;
#is not null 可能走索引
EXPLAIN select * from student where name is not null;
#name字段非空的情况下 判断走name字段索引
#is null 不走索引 优化器优化的效果应该
EXPLAIN select * from student where name is null;
#执行计划显示可能走索引，实际未走
EXPLAIN select * from student where name is not null;
```

总结：当字段值设置为非空的情况下，只有查询 is null 是不走索引的，is not null 是可能走索引的，应该是优化器优化的结果。

**索引效率高的前提是列的区分度大，包含`NULL`的列很难优化，而且对表索引时不会存储 `NULL` 值。如果索引字段可以为 `NULL`，索引的效率会下降。因为它们使得索引、索引的统计信息以及比较运算更加复杂，所以要求列值非空。**

> 1、count(字段NULL)会过滤统计的数据，sum这些函数也会
> 2、使用> < 的时候也会过滤掉为NULL的数据
> 3、group by 的时候会把所有为NULL的数据合并，可以随机生成UUID解决



#### 隐式转换规则

- 如果一个或两个参数都是NULL，比较的结果是NULL，除了NULL安全的<=>相等比较运算符。对于NULL <=> NULL，结果为true。不需要转换
- 如果比较操作中的两个参数都是字符串，则将它们作为字符串进行比较。
- 如果两个参数都是整数，则将它们作为整数进行比较。
- 如果不与数字进行比较，则将十六进制值视为二进制字符串
- 如果其中一个参数是十进制值，则比较取决于另一个参数。 如果另一个参数是十进制或整数值，则将参数与十进制值进行比较，如果另一个参数是浮点值，则将参数与浮点值进行比较
- 如果其中一个参数是TIMESTAMP或DATETIME列，另一个参数是常量，则在执行比较之前将常量转换为时间戳。
- **在所有其他情况下，参数都是作为浮点数（实数）比较的。**

> 案例：
>
> ```sql
> CREATE TABLE t_user_demo (
> id INT(11) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '自增id',
> name VARCHAR(20) DEFAULT NULL COMMENT '姓名',
> phone char(11) DEFAULT NULL COMMENT '手机号',
> age TINYINT DEFAULT NULL COMMENT '年龄',
> sex CHAR(1) DEFAULT NULL COMMENT '性别',
> PRIMARY KEY (id),
> UNIQUE KEY idx_name_phone (name, phone)
> ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
> ```
>
> 针对手机号码字段新增了索引
>
> **针对以下查询，列 phone 由于隐式转换参与了函数计算，不会命中索引，走全表扫描。**
>
> ```sql
> # phone字段是char，查询13933333333是数字类型，则命中隐式转换最后一条规则，作为浮点数比较
> 	select * from t_user_demo 
> 	where name = ‘zz’ and phone = 13933333333;
> #等价于
> 	select * from t_user_demo 
> 	where name = ‘zz’ and CAST(phone AS UNSIGNED) = 13933333333;
> ```
>
> **思考，若 phone 是 int 类型的字段，查询 phone='13933333333' 会不会走索引？**
> 答：会走索引，参考隐式转换规则，索引列上并没有发生函数操作，因此还是可以正常走索引的。
>
> ```sql
> #因为phone是数值类型，执行这样的SQL：
> 	select * from t_user_demo 
> 	where name = ‘zz’ and phone = ‘13933333333’;
> #根据规则，字符串会被作为数值进行比较，所以，实际上会被解析成：
> 	select * from t_user_demo 
> 	where name = ‘zz’ and phone = CAST(‘13933333333’ AS UNSIGNED) ;
> ```



> ### 什么情况走索引，什么情况不走索引？
>
> > 走索引/命中索引 都是指命中对应字段索引
>
> **1、索引列参与计算，不走索引**
>
> > SELECT `username` FROM `t_user` WHERE age=20;-- 会使用索引
> > SELECT `username` FROM `t_user` WHERE age+10=30;-- 不会使用索引！！因为所有索引列参与了计算
> > SELECT `username` FROM `t_user` WHERE age=30-10;-- 会使用索引
>
> **2、索引列使用函数，可能不走索引**
>
> > -- 不会使用索引,因为使用了函数运算,原理与上面相同
> > SELECT username FROM t_user WHERE concat(username,'1') = 'admin1'; 
> > -- 会使用索引
> > SELECT username FROM t_user WHERE username = concat('admin','1'); 
>
> **3、索引列使用 like 语句，可能不走索引**
>
> > SELECT * FROM USER WHERE username LIKE 'mysql测试%'   --走索引
> > SELECT * FROM USER WHERE username LIKE '%mysql测试'   --不命中索引
> > SELECT * FROM USER WHERE username LIKE '%mysql测试%'  --不命中索引
>
> **4、数据类型隐式转换，字符串列与数字直接比较，不走索引**
>
> > -- stock_code字符串类型带索引
> > SELECT * FROM `stock_data` WHERE stock_code = '600538'  --走索引
> > SELECT * FROM `stock_data` WHERE stock_code = 600538  --不走索引
>
> **5、尽量避免 OR 操作，也不一定会走索引！具体可看下执行计划，才能知道走不走**
>
> > -- stock_code带索引，open不带索引
> > SELECT * FROM `stock_data` WHERE `stock_code` = '600538' OR `open` = 6.62  
> > -- stock_code带索引，up_down_pre带索引
> > SELECT * FROM `stock_data` WHERE `stock_code` = '600538' OR `up_down_pre` = 5.1   
>
> **8、索引列使用 in 语句，可能不走索引**
>
> > -- stock_code数据类型为varchar
> > SELECT * FROM `stock_data` WHERE `stock_code` IN ('600538')  -- 走索引
> > SELECT * FROM `stock_data` WHERE `stock_code` IN ('600538','688663','688280')  -- 走索引
> > SELECT * FROM `stock_data` WHERE `stock_code` IN (大量数据)  -- 不走索引
> > SELECT * FROM `stock_data` WHERE `stock_code` IN (600538)  -- 不走索引   因为字段类型是varchar而非int
>



#### 查询每个学生的各科成绩，以及总分和平均分

题目及解答： https://www.jianshu.com/p/ac67d3e2d2de

```sql
select s.code, s.name,
max(case when e.name = '数学' then e.score else 0 end) 数学,
max(case when e.name = '语文' then e.score else 0 end) 语文,
max(case when e.name = '英语' then e.score else 0 end) 英语,
sum(e.score) 总分,
avg(e.score) 平均分
from student s LEFT JOIN exam e on s.`code` = e.code
GROUP BY s.code;
```



#### 有一张表，字段是 班级class 学生id 总成绩score，写出查询各个班级成绩最高分的同学id（每个班级最高分同学可能多个并列）

```sql
select score.* from score inner join (
	select class_name,max(total_score) max from score GROUP BY class_name
) b on score.class_name = b.class_name and score.total_score = b.max
order by class_name,student_id;
```

![image-20220426234221941](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220501212814.png)





## Spring

#### AOP 和 IOC

**AOP：**

> 为了实现Bean的管理和装配，增加一个中间层代理（字节码增强）来实现所有对象的托管。
>
> 对于接口类型：
>
> ​	默认使用 JdkProxy 技术，生成一个代理，使用代理先执行增强操作，再执行原本的方法。如果要使用CGLIB，开启 proxyTargetClass 配置。
>
> 对于非接口类型：
>
> ​	使用 CGLIB 生成一个子类，在子类方法中先执行增强的方法。
>
> ![](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418215139.png)



**IOC - 控制反转**

也称 DI 依赖注入。就是将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理，不用考虑对象是如何被创建出来的。**我们自己在对象中主动控制去直接获取依赖对象，变为交由容器来控制创建及注入依赖对象。**

**好处是对象之间的耦合度或者说依赖程度降低**

 循环依赖的问题如何解决？ A属性依赖B,B属性依赖A

在 Spring 中，类A 和 类B 是先各自被独立创建出来的，在后面对属性进行装配的时候，这时候两个对象的引用都是有的，可以彼此给对方赋值。构造方法中循环依赖就没有办法了。



#### IOC 的底层实现

> 根据需求编写XML文件，配置需要创建的bean。
> 编写程序读取XML文件，获取bean相关信息，类、属性、id。
> 根据第2步获取到的信息，结合反射机制动态创建对象，同时完成属性的赋值。
> 将创建好的bean存入Map集合，设置key-value映射，key就是bean中的id值，value就是bean对象。
> 提供方法从Map通过id获取到对应的value。
>
> 核心代码类似：
>
> ```java
> public class MyClassPathXmlApplicationContext implements ApplicationContext {
> private Map<String,Object> iocMap;
> 
> public MyClassPathXmlApplicationContext(String path) {
>   iocMap = new HashMap<>();
>   //解析XML
>   parseXML(path);
> }
> public void parseXML(String path){
>   SAXReader saxReader = new SAXReader();
>   try {
>       Document document = saxReader.read("src/main/resources/"+path);
>       Element root = document.getRootElement();
>       Iterator<Element> rootIter = root.elementIterator();
>       while (rootIter.hasNext()){
>           Element bean = rootIter.next();
>           String idStr = bean.attributeValue("id");
>           String classStr = bean.attributeValue("class");
>           //反射动态创建对象
>           Class  clazz = Class.forName(classStr);
>           Constructor constructor = clazz.getConstructor();
>           Object object = constructor.newInstance();
>           //给属性赋值
>           Iterator<Element> beanIter = bean.elementIterator();
>           while (beanIter.hasNext()){
>               Element property = beanIter.next();
>               String propertyName = property.attributeValue("name");
>               String propertyValue = property.attributeValue("value");
>               //获取setter方法
>               //num对应setNum，brand对应setBrand
>               String methodName = "set"+propertyName.substring(0,1).toUpperCase() + propertyName.substring(1);
>               //获取属性类型
>               Field field = clazz.getDeclaredField(propertyName);
>               Method method = clazz.getMethod(methodName,field.getType());
>               Object value = propertyValue;
>               //类型转换
>               switch(field.getType().getName()){
>                   case "java.lang.Integer":
>                       value = Integer.parseInt(propertyValue);
>                       break;
>                   case "java.lang.Double":
>                       value = Double.parseDouble(propertyValue);
>               }
>               //调用方法
>               method.invoke(object,value);
>           }
>           //存入Map
>           iocMap.put(idStr,object);
> 
>       }
>   } catch (DocumentException e) {
>       e.printStackTrace();
>   } catch (ClassNotFoundException e){
> 
>   } catch (NoSuchMethodException e) {
>       e.printStackTrace();
>   } catch (InvocationTargetException e) {
>       e.printStackTrace();
>   } catch (InstantiationException e) {
>       e.printStackTrace();
>   } catch (IllegalAccessException e) {
>       e.printStackTrace();
>   } catch (NoSuchFieldException e) {
>       e.printStackTrace();
>   }
> }
> }
> ```



#### Spring Bean 生命周期

1. Bean 容器找到配置文件或注解中 Spring Bean 的定义,Bean 容器利用 Java Reflection API 创建 Bean 的实例
2. Spring 使用依赖注入填充属性
3. 如Bean对应的类 实现 BeanNameAware 接口，则可在实现的方法 setBeanName 参数拿到 bean id
4. 如Bean对应的类 实现 BeanFactoryAware 接口，则可拿到 BeanFactory 实例
5. 如Bean对应的类 实现 ApplicationContextAware 接口，可拿到 ApplicationContext 对象
6. 如定义类实现 BeanPostProcessor 接口，则调用其 Before 方法，从参数中拿到 bean 实例 
7. 如果Bean对应的类实现 InitializingBean 方法，则执行afterPropertiesSet 方法
8. 执行 init-method 指定自定义的方法
9. 执行 BeanPostProcessor 的 After 方法 (AOP 在这一步实现)
10. Bean 的使用
11. 如果类实现 DisposableBean 接口，则执行 destory() 方法
12. 执行 destroy-method 指定的方法

> 为什么这么复杂？
>
> Spring 是对 Bean 管理的基础设施，需要管理各种各样的 Bean，不管是 Pojo 还是 Service，都需要统一的控制起来。

![image-20220418222557859](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418222557.png)

![image-20220418223104351](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418223104.png)



#### Spring 事务

##### 事务可能失效的情况？

- 方法的访问权限需要 public
- 异常被catch
- throw 错误的异常 默认只对出现运行期异常(java.lang.RuntimeException及其子类)，Error进行回滚。 

##### 事务的特性（ACID）

> 原子性（Atomicity）：事务是一个原子操作 ，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
>
> 一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
>
> 隔离性（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
>
> 持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。

##### Spring 事务的七种事务传播行为

**@Transactional(propagation = Propagation.REQUIRED)**

- REQUIRED(Spring默认的事务传播类型)

  > **如果当前没有事务，则自己新建一个事务，如果当前存在事务，则加入这个事务**

- SUPPORTS

  > **当前存在事务，则加入当前事务，如果当前没有事务，就以非事务方法执行**

- MANDATORY

  > 当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常

- REQUIRES_NEW

  > 创建一个新事务，如果存在当前事务，则挂起该事务。

- NOT_SUPPORTED

  > 始终以非事务方式执行,如果当前存在事务，则挂起当前事务

- NEVER

  > 不使用事务，如果当前事务存在，则抛出异常

- NESTED

  > 如果当前事务存在，则在嵌套事务中执行，否则REQUIRED的操作一样（开启一个事务）

**@Transactional(rollbackFor=Throwable.class)** 

> rollbackFor默认值为UncheckedException，包括了RuntimeException和Error。
>
> @Transactional不指定rollbackFor时，Exception及其子类都不会触发回滚 



#### spring循环依赖

参考bean的生命周期，第一步是实例化，不需要加载属性，第二步依赖注入的时候，已经有两个引用了，不会死循环；

具体根据bean的生命周期，可参考：

​	https://www.jianshu.com/p/8666d2ffabfb

​	https://blog.csdn.net/qq_45538657/article/details/120188252

循环依赖问题在Spring中主要有三种情况：

（1）通过构造方法进行依赖注入时产生的循环依赖问题。

（2）通过setter方法进行依赖注入且是在多例（原型）模式下产生的循环依赖问题。

（3）通过setter方法进行依赖注入且是在单例模式下产生的循环依赖问题。

在Spring中，只有第（3）种方式的循环依赖问题被解决了，其他两种方式在遇到循环依赖问题时都会产生异常。这是因为：

第一种构造方法注入的情况下，在new对象的时候就会堵塞住了。

第二种setter方法（多例）的情况下，每一次getBean()时，都会产生一个新的Bean，需要处理依赖，如此反复下去就会有无穷无尽的Bean产生了，最终就会导致OOM问题的出现。

Spring在单例模式下的setter方法依赖注入引起的循环依赖问题，主要是通过二级缓存和三级缓存来解决的，其中三级缓存是主要功臣。解决的核心原理就是：在对象实例化之后，依赖注入之前，Spring提前暴露的Bean实例的引用在第三级缓存中进行存储。

**答：对于属性的循环依赖，主要是通过二级缓存和三级缓存来解决，在 Bean 的生命周期第一步实例化，会将 Bean 实例的引用放到三级缓存中，它是一个 bean 工厂，然后在第二步进行依赖注入的时候，根据需要注入的bean，从一级、二级、三级缓存里面依次查找，找到对应bean 的工厂获取到实例完成注入并添加到二级缓存，完成初始化后对象放入一级缓存。**

**一级缓存（Map<String, Object> singletonObjects）：**用来保存实例化、初始化都完成的bean对象。

**二级缓存（Map<String, Object> earlySingletonObjects）：**用来保存实例化完成，但是未初始化完成的对象（这个对象不一定是原始对象，也有可能是经过AOP生成的代理对象）。

**三级缓存（Map<String, ObjectFactory<?>> singletonFactories）：**用来保存一个对象工厂（ObjectFactory），提供一个匿名内部类，用于创建二级缓存中的对象。

![Spring003](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220503170135.png)

![aa](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220503165156.png)



拓展：**为什么要使用三级缓存呢？二级缓存能解决循环依赖吗？**

只用二级缓存，取出来的可能是原始对象或生产的代理对象，只有先用三级缓存，它存放的是生成具体对象的匿名内部类（ObjectFactory），不区分是原始对象还是代理对象，多态的好处。

如果没有 AOP 代理，二级缓存可以解决问题，但是有 AOP 代理的情况下，只用二级缓存就意味着所有 Bean 在实例化后就要完成 AOP 代理，这样违背了 Spring 设计的原则，Spring 在设计之初就是通过 AnnotationAwareAspectJAutoProxyCreator 这个后置处理器来在 Bean 生命周期的最后一步来完成 AOP 代理，而不是在实例化后就立马进行 AOP 代理。






#### spring中BeanFactory和FactoryBean的区别

BeanFactory是个Factory，也就是IOC容器或对象工厂，FactoryBean是个Bean。

在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的。但对FactoryBean而言，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和装饰器模式类似。

FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现。它们隐藏了实例化一些复杂Bean的细节



#### Autowired注解与Resource注解的区别

**两者的共同点**

1. @Resource注解和@Autowired注解都可以用作bean的注入.
2. 在接口只有一个实现类的时候,两个注解可以互相替换,效果相同.

**两者的不同点**

> **@Resource注解是Java自身的注解,@Autowired注解是Spring的注解.**
> **@Resource注解有两个重要的属性,分别是name和type**,如果name属性有值,则使用byName的自动注入策略,将值作为需要注入bean的名字,如果type有值,则使用byType自动注入策略,将值作为需要注入bean的类型.如果既不指定name也不指定type属性，这时将通过反射机制使用byName自动注入策略。即@Resource注解默认按照名称进行匹配,名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，**默认取字段名，按照名称查找,当找不到与名称匹配的bean时才按照类型进行装配**。但是需要注意的是，如果**name属性一旦指定，就只会按照名称进行装配**。
>
> **@Autowired注解是spring的注解,此注解只根据type进行注入,不会去匹配name.但是如果只根据type无法辨别注入对象时,就需要配合使用@Qualifier注解或者@Primary注解使用.**







### Spring Boot

Spring Boot 是为了简化 Spring 使用的，让开发、配置、运行变得更简单，约定大于配置。

#### Spring Boot 自动配置



#### Spring Boot Starter

参考：https://blog.csdn.net/qq_27828675/article/details/115700493

SpringBoot核心就是几个注解：SpringBootConfiguration、EnableAutoConfiguration、ComponentScan，依赖这几个注解完成了所谓自动装配的功能。（包含在启动类 SpringBootApplication 注解）

作用：通过配置和实现 Starter，可以通过 Spring Boot 启动的时候，拉起一系列的操作，**达到低配置或零配置使用的目的**，如 shardingsphere 构建数据源的逻辑。

SpringBoot程序在启动过程中会解析SpringBootConfiguration、EnableAutoConfiguration、ComponentScan三个注解：

> **SpringBootConfiguration**：包含了Configuration注解，实现配置文件
> **ComponentScan**：指定扫描范围
> **EnableAutoConfiguration**：
>
> > 通过源码可以知道，该注解使用Import引入 AutoConfigurationImportSelector 类，而**AutoConfigurationImportSelector**类通过SpringFactortisLoader加载了所有jar的MATE-INF文件夹下面的**spring.factories**文件，spring.factories包含了所有需要装配的XXXConfiguration类的全限定名。XXXConfiguration类包含了实例化该类需要的信息，比如说如果这是个数据源Configuration类，那么就应该有数据库驱动、用户名、密码等等信息。

主要文件：

> 1. spring.provides
>
>    provides 属性记录了当前 Starter 的名字
>
> 2. spring.factories
>
>    EnableAutoConfiguration 指定启动加载的配置类
>
>    ```properties
>    org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.jjh.work08.StudentAutoConfiguration
>    ```
>
>    指定自动配置的类，记录入口
>
> 3. additional.json
>
>    用于校验、自动提示，可以不配置
>
> 4. 自定义的 Configuration 类
>
>    spring.factories指定的配置类，作为 Starter 入口，构建 Bean



#### Spring Boot 配置类注解

**@ConfigurationProperties(prefix = "jjh.test")** 

> 定义配置文件类，可以通过在 yml 配置对应的属性
>
> ![image-20220419134619849](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220419134707.png)



**@EnableConfigurationProperties**({ConfigBean.class}) 

> 引入配置类，注入到成员属性

**@ConditionalOnProperty(prefix = "jjh.test", name = "enabled", havingValue = "true")**

> 跟 @Configuration 一起使用，声明在该配置成立下 配置类才生效





### Spring Cloud 生态

![image-20220418133039401](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418140623.png)

#### 微服务整体流程

用户请求经过微服务网关（Spring Cloud Getaway），网关调用微服务，服务通过Eureka 注册中心注册发现，配置中心使用 Spring Cloud Config 或 携程的 Apollo，容错限量使用 Hystrix，中间的负载均衡使用 Ribbon，用户鉴权可以使用 Shiro、Spring Security。

外部使用 APM 进行监控，日志监控 ELK , 使用队列 Kafka 统一收集监控相关的数据。

![image-20220418133233039](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418133233.png)

**Spring Cloud Alibaba**

![image-20220418143001999](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418143002.png)

#### 微服务的应用场景

- 业务系统复杂度高，微服务在业务复杂度低的时候生产力反而比单体架构低。
- 随着业务复杂度升高，单体架构的生产力快速下降，微服务相对平稳。
- 拆分本身是非常复杂的。

#### 各组件

##### 配置中心

> Spring Cloud Config
>
> > 使用了 Git Server 作为配置的数据来源，Config Server 去远程仓库去拉，拉取到本地的 Git Server，然后使用本地仓库分发配置信息到各微服务的节点。
> >
> > 当远程仓库发生变化，便会通知到各服务的节点。
> >
> > ![image-20220418134940053](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418134940.png)

##### 注册中心

> Eureka : 服务注册发现
>
> > Eureka 一般部署三个节点的 Server 集群，服务提供者把自己注册到 Eureka 上，如果服务提供者发生变化会 Renew，更新，下线则通知 Consul 到 Eureka 服务取消掉自己的注册。
> >
> > 消费者 先去Eureka 拿到服务的注册信息，通过远程调用，进行服务调用得到结果。
> >
> > ![image-20220418135507467](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418135507.png)
>
> Consul
>
> Zookeeper
>
> > ZooKeeper是一个为分布式应用提供一致性服务的开源组件，它内部是一个分层的文件系统目录树结构，规定同一个目录下只能有一个唯一文件名。
> >
> > 基于ZooKeeper实现分布式锁的步骤如下：
> >
> > （1）创建一个目录mylock；
> > （2）线程A想获取锁就在mylock目录下创建临时顺序节点；
> > （3）获取mylock目录下所有的子节点，然后获取比自己小的兄弟节点，如果不存在，则说明当前线程顺序号最小，获得锁；
> > （4）线程B获取所有节点，判断自己不是最小节点，设置监听比自己次小的节点；
> > （5）线程A处理完，删除自己的节点，线程B监听到变更事件，判断自己是不是最小的节点，如果是则获得锁。

##### 熔断

当出现故 障是不可避免的故障时，停止级联故障并在复杂的分布式系统中实现弹性。

> Hystrix
>
> > 不在维护
>
> Alibaba Sentinel
>
> > 能够做到 限流、过载保护、服务降级
> >
> > ![image-20220418140904813](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418140904.png)
>
> Reslient4j
>
> > 第三方开源的

##### 网关

> Spring Cloud Getaway
>
> > 官方推荐的网关：概念和 Zuul2 类似，基于 Spring5 的 WebFlux 来实现的，WebFlux 底层是 Reactor Netty，还是 Netty 实现的。
>
> Zuul
>
> > 基于 BIO ，性能一般，并发有限
> >
> > ![image-20220418135855508](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418135855.png)
>
> Zuul2  
>
> > 基于 Netty 实现的 NIO 技术
> >
> > ![image-20220418135822524](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418135822.png)

##### RPC 接口封装

> Feign
>
> > 主要功能：帮我们封装远程的REST 接口，变成一个本地的 RPC 调用，生产本地代码（桩）进行调用。
>
> ![image-20220418140223219](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418140223.png)

##### 负载均衡

> Ribbon 
>
> 一般只用于负载均衡



## Redis

#### Redis 是多线程还是单线程？

从整个应用上看，从来都不是单线程的，在 Redis 6之前，处理网络请求这块一直都是 BIO，单线程的，Redis 6 之后处理网络请求变成 NIO 多线程处理。

执行命令、处理内存数据还是单线程的。



#### Redis 基本数据类型

##### 五种基本数据类型

1. 字符串类型（string）

   > 应用场景：
   >
   > 1. 缓存功能：字符串类型可以缓存各种类型格式的数据。
   > 2. 计数器功能，可以用 incr 相关命令进行计数功能。
   > 3. 可以用于共享 session 之类的信息。

   ![image-20220418093506290](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418093640.png)

2. 散列（hash）

   > hset h1 a 11 b 22
   >
   > hget h1 a
   >
   > 应用场景：
   >
   > 1. 适合缓存结构体的信息，如一个对象 可以单独存储对应不同属性，不需要类似字符串类型序列化整个对象进行存储。（缺点是过期功能只能设置在key上，不能设置到每一个具体的field）
   >
   > ![image-20220418094031105](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418094031.png)

   ![image-20220418093727218](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418093727.png)

3. 列表（list）

   > 应用场景：
   >
   > 1. 利用 lpush rpop，实现消息队列的功能。
   > 2. 可以利用队列实现一个秒杀的功能，pop完之后就代表卖完了。

   ![image-20220418094506962](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418094507.png)

   ![image-20220418094146014](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418094146.png)

4. 集合（set)

   > 应用场景：
   >
   > 1. 可以做一些标签去重。

   ![image-20220418094748650](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418094748.png)

   ![image-20220418094627359](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418094627.png)

5. 有序集合（sort set）

   > 应用场景：
   >
   > 1. 可以做一些点赞排行榜，热门歌曲排行。
   > 2. 可以配置不同的分数权重，任务按序执行。

   ![image-20220418095355777](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418095355.png)

   ![image-20220418094904936](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418094905.png)

##### 三种高级数据类型

- Bitmaps 位图 ：可以快速的比较、判断是否存在
- Hyperloglogs：提供不完全精确的去重计数方案
- GEO：判断地理位置经纬度

![image-20220418095835336](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418095835.png)





#### Redis 单线程为什么那么快？

可参考：https://www.cnblogs.com/ludongguoa/p/15338349.html

- 直接操作内存，作为缓存服务，性能瓶颈在内存和网络
- 单线程 反而不需要锁、不需要多线程的上下文切换，节省资源
- IO多路复用器，极大的提高了IO效率

##### IO 多路复用模型

> - IO多路复用器：负责监听多个socket，会将产生事件的 socket 放入队列中排队
>
>   > **同步通信 阻塞IO - IO 复用模型 - 事件驱动模型 【Netty 的 NIO 和零拷贝模型 也叫 Reactor IO】**
>   >
>   > > - 相当于非阻塞式IO 的升级，不需要用户线程一直轮询
>   > >
>   > > - 由内核 select 系统负责请求线程的轮询工作，并可以复用于多个用户线程的请求【select线程阻塞, 不阻塞用户线程】
>   > >
>   > > - 等到内核处理完成后，poll通知用户线程，开始复制到共享空间并返回，即请求成功。【线程阻塞】
>
> - 事件分发器：每次从队列中取出一个 socket，根据 socket 的事件类型交给对应的事件处理器进行处理。
>
> - 事件处理器
>
>   > 连接应答处理器：接收到客户端的连接后，创建socket连接
>   >
>   > 命令请求处理器：解析从socket读取的命令，交由对应的handler执行。
>   >
>   > 命令回复处理器：负责将服务器执行命令后得到的命令回复先写到缓存，再通过socket返回给客户端。
>
> ![img](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220416224931.jpeg)

#### Redis 事务

开启事务后，redis 将命令先写到队列，待执行事务提交。

- 开启事务：multi 
- 命令入队 
- 执行事务：exec 
- 撤销事务：discard

Watch 实现乐观锁 watch 一个 key，发生变化则事务失败

#### Redis Lua脚本

**直接执行**

> 执行脚步 返回字符串 0表示有0个参数
>
> eval "return 'hello world'"0   
>
> > hello world
>
> 执行脚本，设置 11：hh
>
> eval "redis.call('set',KEYS[1],ARGV[1])" 1 11 hh
>
> > get 11
> >
> > "hh"

**预编译**

> script load script "redis.call('set',KEYS[1],ARGV[1])"
>
> > hash1
>
> evalsha hash1 1 11 hh
>
> > get 11
> >
> > "hh"

![image-20220417145105593](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220417145112.png)

#### Redis 备份与恢复

##### RDB文件（默认备份方式）

**备份**

> save 命令 备份文件 在redis 数据目录生成数据文件 dump.rdb 
>
> bgsave 异步执行备份

**数据恢复**

> 将文件放入到redis数据目录下，启动服务即可自动恢复数据
>
> 获取文件目录命令  config get dir

##### AOF 文件备份

配置文件 appendonly 配置为 yes，表示使用 AOF 追加的方式，将执行的命令写入 aof 文件进行备份。

配置追加配置策略：修改 appendfsync :

- always 同步追加，redis 执行一个命令追加一个命令
- everysec 每秒追加一次，将一秒内执行的命令进行追加写入到 aof 文件
- no 不追加

![image-20220417150810390](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220417150810.png)

#### 缓存策略/缓存常见问题

##### **缓存穿透、缓存击穿和缓存雪崩**

缓存和数据库都不存在都key

![img](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220305172125.png)

缓存击穿：

从数据库加载数据更新到缓存操作 加上互斥锁，避免多个线程发现缓存失效都去数据库查询，让一个线程查询，其它线程阻塞等待。

![img](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220305172132.png)

![img](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220305172138.png) 

#### Redis 性能优化

内存优化：

>  hash-max-ziplist-value 64  表示使用带压缩的方式即ziplist 进行存储 hash 数据，压缩内存空间

CPU优化

> 1. 单线程避免阻塞
>
>    避免使用 keys 类似的命令阻塞线程
>
>    > keys 可以使用 scan 指令代替，scan 指令可以无阻塞的提取出指定模式的 key 列表，但是会有一定的重复概率，在客户 端做一次去重就可以了，但是整体所花费的时间会比直接用 keys 指令长。
>
> 2. 谨慎使用范围操作 类似，比较费时的操作，阻塞线程 引起超时
>
> 3. SLOWLOG get 10  查看最近10条比较慢的命令（默认执行超过10ms才会记录）

![image-20220417151206170](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220417151206.png)

#### Redis 分布式锁

锁和代码实现可参考：

​	https://www.cnblogs.com/wangyingshuo/p/14510524.html

​	https://www.jianshu.com/p/47fd7f86c848

​	https://blog.csdn.net/zxd1435513775/article/details/122194202

业务实现可参考：https://blog.csdn.net/qq_45604746/article/details/122058542

借用 redis 单线程处理内存数据的特性，杜绝了并发的问题

关键点：原子性、互斥、超时

**获取锁**

SET dlock my_random_value NX PX 30000

> 设置键为 dlock 的值为my_random_value ，当键不存在才设置，超时30秒自动失效。
>
> NX：当键不存在的时候才设置该值
>
> XX：当键存在的时候才设置该值
>
> PX：过期时间 单位毫秒
>
> EX：过期时间 单位秒

- 设置过期时间，防止应用获取锁之后，宕机未解锁，导致其他机器获取不到锁。

**释放锁**

利用 Redis 单线程的特性和内置的 Lua 引擎，执行 Lua 脚本释放锁。

> ```lua
> if redis.call('get',KEYS[1]) == ARGV[1] then 
>     return redis.call('del',KEYS[1]) 
> else 
>     return 0 
> end
> ```
>
> 首先查看锁的值是否为刚刚设置的值，确认是自己的锁，然后删除掉这个 key 即释放锁。

- redis 单线程 保证了原子性，不存在并发问题
- 谁上的锁谁才能负责解锁，所以需要先比较值，是自己设置的才能释放锁
- 比较值和删除锁的操作都放在脚本中，保证原子性。

**业务实现**

多台机器负载均衡，将上面 redis 锁封装为 获取锁 释放锁两个方法，key 取同的业务标识，value 使用UUID

请求进来之后，根据 key 和 uuid 值 上锁，业务执行完毕，释放锁

UUID 是时间戳、随机数和网卡MAC地址 组合的全局唯一的值，多台机器不存在冲突。

**Redis 集群下会有什么问题，如何保证分布式锁？**

主从节点同步数据存在延迟，当在主节点获取到锁之后，还没来得及同步到从节点，主节点挂了，这个时候新选举出来的主节点是没有set值的，也就是说锁失效了，其它服务器可以拿到锁，依然存在并发问题。

还有业务执行时间过长，且不可控，导致 业务未执行完锁过期释放的问题，需要有守护线程执行自动续期，等到业务执行完才进行释放。

使用`RedLock`的`Redisson`落地实现：

> Redisson 自带 Watch dog 看门狗，可以定时去查看锁状态，延长锁生存时间。

Redlock核心思想是这样的：

> ❝
>
> 搞多个Redis master部署，以保证它们不会同时宕掉。并且这些master节点是完全相互独立的，相互之间不存在数据同步。同时，需要确保在这多个master实例上，是与在Redis单实例，使用相同方法来获取和释放锁。
>
> ❞

简化下步骤就是：

- 按顺序向5个master节点请求加锁
- 根据设置的超时时间来判断，是不是要跳过该master节点。
- 如果大于等于3个节点加锁成功，并且使用的时间小于锁的有效期，即可认定加锁成功啦。
- 如果获取锁失败，解锁！

**Redission 锁在 Redis 中存了锁名称的一个 hash ，hash 中 存的 key 是随机字符串+线程id value 是 1(记录可重入锁当前计数，当前线程每获取一次则加一)**

**Redisson 如果释放锁操作本身异常了，watch dog 还会不停的续期吗？**

不会，在 unlock 方法中会取消续期。

https://blog.csdn.net/qq_26222859/article/details/79645203	

#### Redisson 可重入锁的实现

client.getLock 方法默认创建的就是可重入锁

```java
        Config config = new Config();
        config.useSingleServer().setAddress("redis://127.0.0.1:6379");
        final RedissonClient client = Redisson.create(config);
        RLock lock = client.getLock("lockjjh");
		lock.lock();
```

getLock方法中 根据当前的连接和锁名称创建 RedissionLock 对象并返回。

```java
public RLock getLock(String name) {
    return new RedissonLock(this.connectionManager.getCommandExecutor(), name);
}
```

RedissionLock  的 lock() 方法，就是根据当前线程的 id 去尝试获取锁

![image-20220508152834787](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220508152841.png)

在 Redis 锁中，可重入锁是以 hash 的方式进行存储，每一个 field 存的是当前连接的id+线程id，value 是 1(记录可重入锁当前计数，当前线程每获取一次则加一)

![image-20220508154645557](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220508154645.png)

![image-20220508154632770](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220508154632.png)





#### Redis 集群配置

##### 主从复制

**执行命令方式**

>  #当前redis作为从库，6379作为主库
>
> SlAVEOF 127.0.0.1 6379

**修改配置文件方式**

redis_6380.conf

配置 端口号、pid文件、数据文件目录、主库地址

```
port 6380 										--指定端口
pidfile /var/run/redis_6380.pid 				--pid文件位置
dir /usr/local/redis-6.2.6/data/redis6380 		--数据文件的位置
replicaof ::1 6379 								--配置主库为6379端口的redis库
```

##### sentinel 高可用

sentinel0.conf

> sentinel myid 8d992c54df8f8677b0b345825f61fb733c73d14c
> #配置master节点，2表示至少需要两个sentinel选举才能决定主库
> sentinel monitor mymaster 127.0.0.1 6379 2
> #每10秒监听Redis的状态，如果10秒内不回应则会触发选举
> sentinel down-after-milliseconds mymaster 10000
> #做failover允许最大的时间，默认3分钟  3分钟内选举出一个主库，如果失败了就再去选举另一个
> #sentinel failover-timeout mymaster 180000
> #单位时间只有1个从库拉取主库的数据
> #sentinel parallel-syncs mymaster 1
> #端口
> port 26379

启动3个redis节点，端口分别为 6379、6380、6381，主库端口6379，使用如下命令启动哨兵

 ./redis-sentinel ../conf/sentinel0.conf

 ./redis-sentinel ../conf/sentinel1.conf

redis节点启动后，停用6379端口节点，自动选举出了6381作为主库；

![image-20220315000916013](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220417221159.png)

当6379重连的时候，被自动作为6380的从库

![image-20220315001057211](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220417221159.png)

重连后不会同步之前的数据

##### Cluster 集群

redis配置文件配置 cluster-enabled yes



#### Zookeeper 分布式锁原理

ZooKeeper是一个为分布式应用提供一致性服务的开源组件，它内部是一个分层的文件系统目录树结构，规定同一个目录下只能有一个唯一文件名。基于ZooKeeper实现分布式锁的步骤如下：

（1）创建一个目录mylock；
（2）线程A想获取锁就在mylock目录下创建临时顺序节点；
（3）获取mylock目录下所有的子节点，然后获取比自己小的兄弟节点，如果不存在，则说明当前线程顺序号最小，获得锁；
（4）线程B获取所有节点，判断自己不是最小节点，设置监听比自己次小的节点；
（5）线程A处理完，删除自己的节点，线程B监听到变更事件，判断自己是不是最小的节点，如果是则获得锁。



#### TCC 三阶段

TCC方案: 分为try（业务预处理）-confirm（业务确认）-cancel（业务取消，回滚try的处理） 三个阶段。

1. 准备操作 Try

   > 完成所有业务检查，预留必须的业务资源

2. 确认操作 Confirm

   > 真正执行的业务逻辑，不做任何业务检查，只使用 Try 阶段预留的业务资源。
   >
   > 因此，只要 Try 操作成功，Confirm 必须能成功，另外，Confirm 操作需要满足幂等性，保证一笔分布式事务能且只能成功一次。

3. 取消操作 Cancel

   > 释放 Try 阶段预留的业务资源。同样，Cancel 操作需要满足幂等性。



#### 如何保证幂等

参考：https://www.cnblogs.com/huigelaile/p/15843021.html#tid-QJW6ZX

在我们分布式系统中，可能会因为网络波动等无法避免的原因，导致请求多次，例如支付系统多次点击，如何保证只会支付一次，避免用户多次支付，这就是我们所说的业务的幂等性。

**一、唯一性约束**

核心业务字段设置为唯一索引，多次插入就会报错，从而保证幂等性。

**二、业务流转状态判断**

状态流转也是最常见的幂等性保证手段。

> 配送业务中，骑手的操作肯定会对业务流转状态进行校验。如果骑手已经提货，那就肯定不能再次提货的。

**三、利用 token 判断去重**

生成一个业务token，写入到 Redis或数据库，并绑定到请求页面，点击提交时携带token发送请求, 业务处理的时候首先判断token是否存在，存在则处理业务，处理完后删除token，如重复请求 下一次进来发现token不存在则不进行处理。页面提交按钮也可以点击之后禁用或跳转新页面

#### 如何保障分布式下原子性？

分布式事务，业务侧可以做 TCC 柔性事务来保障事务，要么一起成功要么一起失败。



### 面试题

#### **Redisson 如果释放锁操作本身异常了，watch dog 还会不停的续期吗？**

不会，在 unlock 方法中会取消续期。

https://blog.csdn.net/qq_26222859/article/details/79645203	

#### 如何模拟缓存穿透

> 可以编写一个接口，请求这个接口的时候查询数据库， 查询缓存和数据库中都没有的 id ，可以用for循环不断创建查询任务添加到线程池中，来模拟大量的并发查询。



#### 缓存击穿，采用以缓存为主，如何保证缓存和数据库同步？

参考：https://blog.csdn.net/whirlwind526/article/details/121206384

强一致性同步成本太高，如果追求强一致，那么没必要用缓存了，直接用mysql即可。通常考虑的，都是最终一致性。

#### 缓存同步解决方案

**方案一**

>  通过key的过期时间，mysql更新时，redis不更新。
>
>  这种方式实现简单，但不一致的时间会很长。如果读请求非常频繁，且过期时间比较长，则会产生很多长期的脏数据。
>
>  优点：
>
>  开发成本低，易于实现；
>
>  管理成本低，出问题的概率会比较小。
>
>  不足：
>
>  完全依赖过期时间，时间太短容易缓存频繁失效，太长容易有长时间更新延迟（不一致）



**方案二**

>  在方案一的基础上扩展，通过key的过期时间兜底，并且，在更新mysql时，同时删除redis缓存。
>
>  ![图片](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220422104057.png)
>
>  优点：
>
>  相对方案一，更新延迟更小。
>
>  不足：
>
>  如果更新mysql成功，更新redis却失败，就退化到了方案一；
>
>  在高并发场景，业务server需要和mysql,redis同时进行连接。这样是损耗双倍的连接资源，容易造成连接数过多的问题。
>
>  



**方案三**

> 针对方案二的同步写redis进行优化，增加消息队列，将redis更新操作交给kafka，由消息队列保证可靠性，再搭建一个消费服务，来异步删除redis缓存。
>
> ![图片](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220422104148.png)
>
> 优点：
>
> 消息队列可以用一个句柄，很多消息队列客户端还支持本地缓存发送，有效解决了方案二连接数过多的问题；
>
> 使用消息队列，实现了逻辑上的解耦；
>
> 消息队列本身具有可靠性，通过手动提交等手段，可以至少一次消费到redis。
>
> 不足：
>
> 依旧解决不了时序性问题，如果多台业务服务器分别处理针对同一行数据的两条请求，举个栗子，a = 1；a = 5; 如果mysql中是第一条先执行，而进入kafka的顺序是第二条先执行，那么数据就会产生不一致。
>
> 引入了消息队列，同时要增加服务消费消息，成本较高，还有重复消费的风险。

**方案四**

> 通过订阅binlog来更新redis，把我们搭建的消费服务，作为mysql的一个slave，订阅binlog，解析出更新内容，再删除核对缓存。
>
> ![图片](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220422104324.png)
>
> 优点：
>
> 在mysql压力不大情况下，延迟较低；
>
> 和业务完全解耦；
>
> 解决了时序性问题。
>
> 缺点：
>
> 要单独搭建一个同步服务，并且引入binlog同步机制，成本较大。

**总结**
方案选型

> 首先确认产品上对延迟性的要求，如果要求极高，且数据有可能变化，别用缓存。
>
> 通常来说，方案1就够了，笔者咨询过4，5个团队，基本都是用方案1，因为能用缓存方案，通常是读多写少场景，同时业务上对延迟具有一定的包容性。方案1没有开发成本，其实比较实用。
>
> 如果想增加更新时的即时性，就选择方案2，不过没必要做重试保证之类的。
>
> 方案3，方案4针对于对延时要求比较高业务，一个是推模式，一个是拉模式，而方案4具备更强的可靠性，既然都愿意花功夫做处理消息的逻辑，不如一步到位，用方案4。

**结论**
一般情况，方案1够用。若延时要求高，直接选择方案4。如果是面试场景，从简单讲到复杂，面试官会一步一步追问，咱们就一点点推导，宾主尽欢。

**为什么是删除，而不是更新缓存？**

我们以**先更新数据库，再删除缓存**来举例。

如果是更新的话，那就是**先更新数据库，再更新缓存**。

举个例子：如果数据库1小时内更新了1000次，那么缓存也要更新1000次，但是这个缓存可能在1小时内只被读取了1次，那么这1000次的更新有必要吗？

反过来，如果是删除的话，就算数据库更新了1000次，那么也只是做了1次缓存删除，只有当缓存真正被读取的时候才去数据库加载更新到缓存。

对于实时性要求不高的，等待缓存失效即可，实时性要求高的，可以在更新数据库的时候去删除缓存，等待下一次查询请求更新结果到缓存（如果担心并发查询量大，删除缓存导致缓存击穿问题，在查询的时候没查到redis，查数据库这步加锁，然后只有一个线程去查询并更新索引，其他线程等待，不会给数据库太大压力）。





## 消息队列

#### 队列有什么好处

> - 异步通信：减少线程等待，特别是处理批量等大事务，耗时操作。
> - 系统解耦：系统不直接调用，降低依赖，特别是不在线也能报错通信最终完成。
> - 削峰平谷：压力大的时候，缓存部分请求消息，类似于背压处理。
> - 可靠通信：提供多种消息模式，服务质量，顺序保障等。
>
> ![image-20220425203226084](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220425235830.png)

#### Kafka 的架构说一下？ 

整个架构中包括三个角色。 

- 生产者（Producer）：消息和数据生产者。 
- 代理（Broker）：缓存代理，Kafka 的核心功能。 
- 消费者（Consumer）：消息和数据消费者。 

Kafka 给 Producer 和 Consumer 提供注册的接口，数据从 Producer 发送到 Broker， Broker 承担一个中间缓存和分发的作用，负责分发注册到系统中的 Consumer。

#### Kafka 怎么保证消息是有序的？ 

消息在被追加到 Partition(分区)的时候都会分配一个特定的偏移量（offset）。Kafka 通过偏 移量（offset）来保证消息在分区内的顺序性。发送消息的时候指定 key/Partition

#### Kafka 如何保证不被重复消费？

- 生产者发送每条数据的时候，里面加一个全局唯一的 id，消费到了之后，先根据这个 id 去比如 Redis 里查一下，之前消费过吗，如果没有消费过，就处理，然后这个 id 写 Redis。如果消费过就别处理了。

- 基于数据库的唯一键来保证重复数据不会重复插入多条。因为有唯一键约束了，重复数据 插入只会报错，不会导致数据库中出现脏数据

#### Kafka 怎么保证消息不丢失？

**生产者消息保证不丢失**

> - 使用`producer.send(msg, callback)`带有回调的`send`方法，再回调方法中处理失败的情况
> - 配置失败重试次数retries为一个较大值，且需要设置合适的重试间隔时间，不能太小

**消费者保证消息**

> - 手动提交 offset
>
>   > 当消费者拉取到了分区的某个消息之后，消费者会自动提交了 offset。自动提交的话会有一个 问题，试想一下，当消费者刚拿到这个消息准备进行真正消费的时候，突然挂掉了，消息实际上并没有被消费，但是 offset 却被自动提交了。 解决办法也比较粗暴，我们手动关闭自动提交 offset，每次在真正消费完消息之后再自己手 动提交 offset 。但可能会offset没来得及提交，消费者重启了，导致重复消费，可以业务上做限制。

**Kafka 宕机导致消息丢失**

> 试想一种情况：假如 leader 副本所在的 broker 突然挂掉，那么就要从 follower 副本重新选 出一个 leader ，但是 leader 的数据还有一些没有被 follower 副本的同步的话，就会造成消 息丢失。 
>
> 配置了 unclean.leader.election.enable = false，当 leader 副本发生故障时就 不会从 follower 副本中和 leader 同步程度达不到要求的副本中选择出 leader ，这样降低了 消息丢失的可能性。



#### Kafka 多副本

每个topic 里面有多个 partition, 每个partition 可能是主节点，副本在其他的topic中。

参与选举需要 3副本2确认，每次写入的时候至少有两个副本确认写入了才返回写入成功；类似的还有5副本3确认，作为确认的节点数要大于一半向上取整，否则可能选举出多个主节点导致脑裂。

> #查看 kafka 当前 testk topic 的情况
>
> bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic testk
>
> 如下：
>
> topic testk 中有 3 个 partition
>
> replicationFactor: 副本因子 2 表示2个副本一主一从
>
> Configs:
>
> >  Partition：partition序号 
> >
> >  Leader: partition主节点在哪个序号的服务器（broker) 
> >
> >  Replicas: 2个副本分别在哪个broker上
> >
> >  Isr: 目前处于同步状态的partition
>
> ![](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220419194218.png)

![image-20220419191833352](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220419191833.png)

默认要n/2+1个副本写入成功 认为写入成功。

![image-20220419201300798](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220419201300.png)

#### 什么是高性能？

- 高并发
- 高吞吐量
- 低延迟

#### Kafka 重平衡

参考：https://blog.csdn.net/lisyee/article/details/120896688

##### 重平衡触发条件

重平衡的触发条件有三个：

- 消费者组内成员发生变更：比如组内有机器崩溃离场，或者是添加新的机器入组。
- 订阅主题数量发生变化：kafka目前只支持增加分区。
- 订阅主题的分区数发生变化。

##### 重平衡策略

**Range 平均分配**

> 组内的消费者会**基于topic平均分配** topic 中的分区。如果主题中的分区分区不能平均分配给组内每个消费者，那么某个消费者会分配到额外的分区。
> 假如目前有两个消费者1和消费者2，topicA中有三个分区t1,t2,t3，topicB中有三个分区a1,a2,a3。
> rebalance后的结果：
>
> 消费者1：t1, t2,a1,a2
> 消费者2：t3,a3

**RoundRobin 轮询分配**

> RoundRobin是**基于全部主题的分区**来进行分配的，这是是kafka默认的rebalance分区策略。还是用刚刚的例子来看，
> 举假如目前有两个消费者1和消费者2，topicA中有三个分区t1,t2,t3，topicB中有三个分区a1,a2,a3。
> rebalance后的结果：
>
> 消费者1：t1, t2, 12
> 消费者2：t3, a1, a2
> 可以看到轮询策略可以更加均衡的分配分区给到消费者。

**Sticky 粘性分配**

> 见参考

##### 避免重平衡

> 要说完全避免重平衡，那是不可能的，因为你无法完全保证消费者不会故障。而消费者故障其实也是最常见的引发重平衡的地方，所以要尽量避免重平衡。
>
> 在实际场景中，有部分情况会让kafka错误地认为一个正常的消费者已经挂掉了，我们需要避免这样的情况出现。
>
> 为什么会出现这种情况？
> 在分布式系统中，通常是通过心跳来维持分布式系统的，某些场景由于网络问题没接收到心跳，导致kafka认为消费者挂了，而重新开始重平衡。
>
> **kafka提供了一个参数：session.timout.ms 。这个参数是用来判断多少时间内没有心跳请求的消费者，我们认为他是真正挂了。**
>
> **另一个参数：heartbeat.interval.ms。这个参数控制发送心跳的频率，频率越高越不容易被误判，当然也会消耗更多的cpu资源。**

#### RabbitMQ死信队列？

一般来说，producer将消息投递到queue中，consumer从queue取出消息进行消费，但某些时候由于特定的原因导致queue中的某些消息无法被消费，这样的消息如果没有后续的处理，就变成了死信(Dead Letter)，所有的死信都会放到死信队列中。

“死信”消息会被RabbitMQ进行特殊处理，如果配置了死信队列信息，那么该消息将会被丢进死信队列中，如果没有配置，则该消息将会被丢弃。

**死信队列的来源**

> - 消息被拒绝（basic.reject或basic.nack）并且requeue=false.
> - 消息TTL过期
> - 队列达到最大长度（队列满了，无法再添加数据到mq中）

### 面试题

#### 队列有什么好处

> - 异步通信：减少线程等待，特别是处理批量等大事务，耗时操作。
> - 系统解耦：系统不直接调用，降低依赖，特别是不在线也能报错通信最终完成。
> - 削峰平谷：压力大的时候，缓存部分请求消息，类似于背压处理。
> - 可靠通信：提供多种消息模式，服务质量，顺序保障等。

![image-20220425203226084](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220425203226.png)



####　kafka发生大量消息堆积怎么办？

提高消费者消费能力：可尝试扩容。

> 1 增大partion数量，
> 2 消费者加了并发，服务， 扩大消费线程
> 3 增加消费组服务数量
> 4 kafka单机升级成了集群
> 5 避免消费者消费消息时间过长，导致超时
> 6 使Kafka分区之间的数据均匀分布
>
> 场景：
> 1 如果是Kafka消费能力不足，则可以考虑增加 topic 的 partition 的个数，
> 同时提升消费者组的消费者数量，消费数 = 分区数 （二者缺一不可）
>
> 2 若是下游数据处理不及时，则提高每批次拉取的数量。批次拉取数量过少
> （拉取数据/处理时间 < 生产速度），使处理的数据小于生产的数据，也会造成数据积压
> 
> 



#### 当节点宕机了，生产者发生的消息会丢失吗？

可以部署集群，节点宕机了，自动选举副本作为新的leader。

#### Kafka 为什么这么快？

https://blog.csdn.net/Java0258/article/details/107663998

##### 一、**利用 Partition 实现并行处理**

> 一方面，由于不同 Partition 可位于不同机器，因此可以充分利用集群优势，实现机器间的并行处理。另一方面，由于 Partition 在物理上对应一个文件夹，即使多个 Partition 位于同一个节点，也可通过配置让同一节点上的不同 Partition 置于不同的磁盘上，从而实现磁盘间的并行处理，充分发挥多磁盘的优势。

##### 二、顺序 I/O 写磁盘

> **线性写磁盘**
> 因为硬盘是机械结构，每次读写都会寻址->写入，其中寻址是一个“机械动作”，它是最耗时的。所以硬盘最讨厌随机I/O，最喜欢顺序I/O。为了提高读写硬盘的速度，Kafka就是使用顺序I/O。
>
> **Kafka 中每个分区是一个有序的，不可变的消息序列**，新的消息不断追加到 partition 的末尾，这个就是顺序写。

##### 三、批处理和数据压缩

在很多情况下，系统的瓶颈不是 CPU 或磁盘，而是网络IO。

因此，除了操作系统提供的低级批处理之外，Kafka 的客户端和 broker 还会在通过网络发送数据之前，在一个批处理中累积多条记录 (包括读和写)。记录的批处理分摊了网络往返的开销，使用了更大的数据包从而提高了带宽利用率。

Producer 可将数据压缩后发送给 broker，从而减少网络传输代价，目前支持的压缩算法有：Snappy、Gzip、LZ4。数据压缩一般都是和批处理配套使用来作为优化手段的。

##### 四、 **充分利用 Page Cache**

> 详见参考链接：https://blog.csdn.net/Java0258/article/details/107663998

##### **五、零拷贝技术**

传统模式下，当需要对一个消息进行传输的时候，其具体流程细节如下：

1. 首先通过 DMA copy 将网络数据拷贝到内核态 Socket Buffer
2. 然后应用程序将内核态 Buffer 数据读入用户态（CPU copy）
3. 接着用户程序将用户态 Buffer 再拷贝到内核态（CPU copy）
4. 最后通过 DMA copy 将数据拷贝到磁盘文件

在这个过程当中，文件数据实际上是经过了四次copy操作：

> 硬盘—>内核buf—>用户buf—>socket相关缓冲区—>协议引擎
>
> ![Kafka为什么能那么快的6个原因](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220422162009.png)

###### 写数据 Producer 到 Broker

**Memory Mapped Files**

即便是顺序写入硬盘，硬盘的访问速度还是不可能追上内存。所以Kafka的数据并 不是实时的写入硬盘 ，它充分利用了现代操作系统 分页存储 来利用内存提高I/O效率。

Memory Mapped Files(后面简称mmap)也被翻译成 内存映射文件 ，在64位操作系统中一般可以表示20G的数据文件，它的工作原理是直接**利用操作系统的Page来实现文件到物理内存的直接映射。完成映射之后你对物理内存的操作会被同步到硬盘上。**

而内存映射则减少了两次的 CPU copy 操作

>  减少了 从 Socket 缓冲区拷贝到用户空间 从用户空间拷贝到内核缓存区的两个步骤。
>
>  使用内存映射文件，等待操作系统将拷贝到内核缓冲区的数据拷贝至用户线程。
>
>  > mmap 也有一个很明显的缺陷——不可靠，写到 mmap 中的数据并没有被真正的写到硬盘，操作系统会在程序主动调用 flush 的时候才把数据真正的写到硬盘。Kafka 提供了一个参数——producer.type 来控制是不是主动flush；如果 Kafka 写入到 mmap 之后就立即 flush 然后再返回 Producer 叫同步(sync)；写入 mmap 之后立即返回 Producer 不调用 flush 就叫异步(async)，默认是 sync。

![Kafka为什么能那么快的6个原因](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220422162103.png)

###### 读取数据 Broker 到 Consumer

**零拷贝技术**

> Linux 2.4+ 内核通过 sendfile 系统调用，提供了零拷贝。
>
> 从磁盘读取的消息，直接从内核缓冲区拷贝到网卡，全程不需要 CPU 参与拷贝。
>
> Kafka 在这里采用的方案是通过 NIO 的 transferTo/transferFrom 调用操作系统的 sendfile 实现零拷贝。总共发生 2 次内核数据拷贝、2 次上下文切换和一次系统调用，消除了 CPU 数据拷贝
>
> > “DMA（Direct Memory Access）：直接存储器访问。DMA 是一种无需 CPU 的参与，让外设和系统内存之间进行双向数据传输的硬件机制。使用 DMA 可以使系统 CPU 从实际的 I/O 数据传输过程中摆脱出来，从而大大提高系统的吞吐率。

![Kafka为什么能那么快的6个原因](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220425174201.png)

##### 小总结 

partition 并行处理
顺序写磁盘，充分利用磁盘特性
利用了现代操作系统分页存储 Page Cache 来利用内存提高 I/O 效率
采用了零拷贝技术
Producer 生产的数据持久化到 broker，采用 mmap 文件映射，实现顺序的快速写入
Customer 从 broker 读取数据，采用 sendfile，将磁盘文件读到 OS 内核缓冲区后，转到 NIC buffer进行网络发送，减少 CPU 消耗

> 套接字(Socket)，就是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。一个套接字就是网络上进程通信的一端，提供了应用层进程利用网络协议交换数据的机制。





## 设计模式

### 工厂模式

以往我们创建对象，都需要自己去 new ,自己实现细节

工厂模式 将创建对象的具体过程屏蔽隔离起来，达到提高灵活性的目的。

### 抽象工厂模式

相当于工厂的工厂

### 建造者模式

建造者模式（Builder Pattern）使用多个简单的对象一步一步构建成一个复杂的对象。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

一个 Builder 类会一步一步构造最终的对象。该 Builder 类是独立于其他对象的。

### 责任链模式

顾名思义，责任链模式（Chain of Responsibility Pattern）为请求创建了一个接收者对象的链。这种模式给予请求的类型，对请求的发送者和接收者进行解耦。这种类型的设计模式属于行为型模式。

在这种模式中，通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。

### 适配器模式

Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中 也是用到了适配器模式适配 Controller

AOP 就是通过生成一个类的代理子类，定义了新的方法，在里面再去调用被代理类的方法。代理子类就是类适配器。

缺点：可能会增加系统的复杂性

如下图，定义一个环绕通知的方法，在里面调用原方法的具体执行。

![image-20220418201142069](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220418201142.png)

#### 和装饰器不同之处

- 适配器主要是适配不同的方法，在方法中调用原来被代理的类。装饰器方法是相同的，只是方法中添加了新的功能。

### 装饰器模式

装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。

这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。

我们通过下面的实例来演示装饰器模式的用法。其中，我们将把一个形状装饰上不同的颜色，同时又不改变形状类。

### 模板方法模式

在面向对象程序设计过程中，程序员常常会遇到这种情况：设计一个系统时知道了算法所需的关键步骤，而且确定了这些步骤的执行顺序，但某些步骤的具体实现还未知，或者说某些步骤的实现与具体的环境相关。

例如，去银行办理业务一般要经过以下4个流程：取号、排队、办理具体业务、对银行工作人员进行评分等，其中取号、排队和对银行工作人员进行评分的业务对每个客户是一样的，可以在父类中实现，但是办理具体业务却因人而异，它可能是存款、取款或者转账等，可以延迟到子类中实现。

抽象模板类，负责给出一个算法的轮廓和骨架。它由一个模板方法和若干个基本方法构成。这些方法的定义如下。

① 模板方法：定义了算法的骨架，按某种顺序调用其包含的基本方法。

② 基本方法：是整个算法中的一个步骤，包含以下几种类型。

- 抽象方法：在抽象类中声明，由具体子类实现。
- 具体方法：在抽象类中已经实现，在具体子类中可以继承或重写它。
- 钩子方法：在抽象类中已经实现，包括用于判断的逻辑方法和需要子类重写的空方法两种。

**在接口中已经实现好了固定的方法，安装某种顺序调用其它具体方法或抽象方法，子类实现接口，需要自行实现抽象方法。**

比如 Spring 中的 JdbcTemplate 就是使用了模板方法，正常查询 jdbc 需要：

1、获取connection 
2、获取statement 
3、获取resultset 
4、遍历resultset并封装成集合 
5、依次关闭connection,statement,resultset，而且还要考虑各种异常 

那么可以把

```java
public class TemplateMethodPattern {
    public static void main(String[] args) {
        AbstractClass tm = new ConcreteClass();
        tm.TemplateMethod();
    }
}
//抽象类
abstract class AbstractClass {
    //模板方法
    public void TemplateMethod() {
        SpecificMethod();
        abstractMethod1();
        abstractMethod2();
    }
    //具体方法
    public void SpecificMethod() {
        System.out.println("抽象类中的具体方法被调用...");
    }
    //抽象方法1
    public abstract void abstractMethod1();
    //抽象方法2
    public abstract void abstractMethod2();
}
//具体子类
class ConcreteClass extends AbstractClass {
    public void abstractMethod1() {
        System.out.println("抽象方法1的实现被调用...");
    }
    public void abstractMethod2() {
        System.out.println("抽象方法2的实现被调用...");
    }
}
```





## 网络编程 

### 三次握手和四次挥手

https://www.jianshu.com/p/d3725391af59

![image-20220412151528116](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412151535.png)

![image-20220412152243474](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412152243.png)

#### TCP 和 UDP 区别？ 

- TCP 基于连接，UDP 基于无连接。 
- TCP 要求系统资源较多，UDP 较少。
- UDP 程序结构较简单。 
- TCP 保证数据正确性，UDP 可能丢包。 
- TCP 保证数据顺序，UDP 不保证。

#### 描述下 TCP 连接 4 次挥手的过程？为什么要 4 次挥手？ 

因为 TCP 是全双工，每个方向都必须进行单独关闭。关闭连接时，当 Server 端收到 FIN 报文时，很可能并不会立即关闭 SOCKET，所以只能先回复一个 ACK 报文，告诉 Client 端，”你发的 FIN 报文我收到了”。只有等到 Server 端所有的报文都发送完了，我才能发 送 FIN 报文，因此不能一起发送。故需要四步握手。





## 分布式

#### 什么是CAP？

 一致性（Consistency, C）、可用性（Availability, A）、分区容错性（Partition Tolerance, P）

#### 什么是BASE？和CAP什么区别？

BASE是Basically Available（基本可用）、Soft state（软状态）和Eventually consistent（最终一致性）三个短语的简写，BASE是对CAP中一致性和可用性权衡的结果



## NIO技术 Netty框架

#### 五种IO模型

![image-20220412153352823](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412153352.png)

在通信模式和线程处理模式两个维度，有五种 IO 模型：

![image-20220412153701590](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412153701.png)

1. 同步通信 阻塞式 IO 模型 - BIO

   > -  用户线程发送请求，需要等待内核处理完成（调用网卡，发送socket，拿到数据，拷贝到用户空间）
   >
   > -  等到内核处理完成返回数据才能结束阻塞状态
   >
   > ![image-20220412160257865](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412160257.png)

2. 同步通信 非阻塞式 IO 

   > - 和BIO 相比，用户线程发送请求后，内核会立即返回，告诉用户线程数据没准备好；
   >
   > - 用户线程可以一直轮询，等待内核准备好数据。【未阻塞用户线程】
   >
   > - 等到内核处理完成后，开始复制到用户空间并返回，即请求成功。【线程阻塞】
   >
   > ![image-20220412160240766](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412160240.png)

3. **同步通信 阻塞IO - IO 复用模型 - 事件驱动模型 【Netty 的 NIO 和零拷贝模型 也叫 Reactor IO】**

   > - 相当于非阻塞式IO 的升级，不需要用户线程一直轮询
   >
   > - 由内核 select 系统负责请求线程的轮询工作，并可以复用于多个用户线程的请求【select线程阻塞, 不阻塞用户线程】
   >
   > - 等到内核处理完成后，poll通知用户线程，开始复制到共享空间并返回，即请求成功。【线程阻塞】
   >
   > **注意：内核不需要再复制数据到用户空间，内核与用户空间共享一块内存，实现零拷贝。**
   >
   > ![image-20220412160222889](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412160222.png)

4. 同步通信 非阻塞式IO - 信号驱动IO

   > - 用户进行需要等待数据的时候，会先发送一个信号；等到内核准备好，再通知用户；
   > - 用户再发生请求去读取数据
   > - 数据从内核复制到用户空间
   >
   > 和事件驱动IO的区别：
   >
   > 1. 信号驱动 IO 第一块内核准备数据时是非阻塞的，事件驱动 IO 的 select 系统是存在阻塞的。
   > 2. 事件驱动 IO ，是内核与用户空间共享一块内存，实现零拷贝。
   >
   > ![image-20220412163404032](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412163404.png)

5. 异步通信 非阻塞式 IO

   > 类似信号驱动 IO，但是是等到内核准备好数据即IO执行完毕，再进行通知用户线程读取数据。
   >
   > 常用的 linux 内核不支持
   >
   > ![image-20220412164931376](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412164931.png)

#### Netty 框架

作为网络应用开发的首选框架，如 Spring 5 WebFlux、Zuul、Spring Cloud GateWay（基于WebFlux）。

零拷贝：不需要把用户空间和内核空间的缓冲区数据来回进行拷贝，直接共用一块内存。

三个特点：

> 1. 异步
> 2. 事件驱动
> 3. NIO
>
> 优点：
>
> ![image-20220412171033348](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412171033.png)

Netty 内部的设计实现有三大模块：

> - Netty 核心
>
>   > 1. 高效的数据结构 Byte Buffer，可动态扩容的Buffer
>   > 2. 网络通信的 API
>   > 3. 事件模型
>
> - 传输服务
>
>   > 1. Socket TCP、UDP
>   > 2. HTTP通道
>   > 3. 管道
>
> - 协议支持
>
> ![](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412170459.png)



#### Netty 如何实现高性能

Reactor 模型

![image-20220412202845206](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412202845.png)

**Netty 主要支持三种模型：**

1. Reactor 单线程模型

   > ![image-20220412215308006](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412215308.png)

2. Reactor 多线程模型

   > Reacctor 负责处理请求事件监听，业务这边引入了 worker 线程池进行处理，线程池负责读取不同channel准备好的数据，调用handler进行业务处理。
   >
   > ![image-20220412215351215](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412215351.png)
   >
   > ![image-20220412215848000](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412215848.png)

3. Reactor 主从模型

   > 具体分为三大块：
   >
   > - 所有客户端请求的连接和维护由 mainReactor 进行处理
   > - 当内核数据准备好了，交由 subReactor 负责事件分发处理
   > - 线程池负责读取不同channel准备好的数据，调用handler进行业务处理。
   >
   > ![image-20220412215945498](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412215945.png)
   >
   > 
   >
   > ![image-20220412204523953](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412220210.png)



#### Netty 网络程序优化

![img](https://static001.geekbang.org/infoq/73/734a6c746c7b6f78cff98a6aa2eb05cd.png)





![img](https://static001.geekbang.org/infoq/0c/0cdf6a069e1b4ee1fc6922aa3c44dd56.png)

Nagle算法，两种条件达到一种都会发送数据，如果小包需要每次都发送，打开TCP_NODELAY配置即可。

最大传输数据单元（MTU）: 默认为1500 Byte

最大传输数据包: 默认1500 Byte - 20 Byte（IP头）- 20 Byte（TCP头） 即 1460 Byte

#### API 网关的主要功能

![img](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220412214307.png)





## linux 基本命令 

#### 查看服务器 磁盘、内存空间

df -h 查看磁盘

free -h 查看内存

top -h 查看排名 

#### grep 搜寻命令

参考：https://www.cnblogs.com/lianstyle/p/8572227.html

[root@www ~]# grep [-acinv] [--color=auto] '搜寻字符串' filename
选项与参数：
-a ：将 binary 文件以 text 文件的方式搜寻数据
-c ：计算找到 '搜寻字符串' 的次数
-i ：忽略大小写的不同，所以大小写视为相同
-n ：顺便输出行号
-v ：反向选择，亦即显示出没有 '搜寻字符串' 内容的那一行！
--color=auto ：可以将找到的关键词部分加上颜色的显示喔！

```
#查找passwd文件中 root 所在，将显示所在那一行的内容 -n显示行号
grep -n root /etc/passwd
#查找passwd文件中 无 root 所在，将显示没有root的行
grep -v root /etc/passwd
#将dmesg输出的信息  用grep搜索，显示行号，显示关键字 eth 后三行，前两行
dmesg | grep -n -A3 -B2 --color=auto 'eth'
#在当前目录搜索所有文件中包括‘http’字符的文件，列出那一行的内容
grep -n http * 
#-r表示递归查找，查找/opt 下所有文件 包括‘http’字符的文件，列出那一行的内容，如果不加-r之会在给出文件路径那个位置查找
grep -rn http /opt 
# -l 只显示查找到的文件，不显示查找到的那行内容
grep -lrn http /opt 

# 正则表达式
# 搜寻 test 或 taste 这两个单词
grep -n t[ae]st regular_express.txt
# 搜索有oo的行，但不想要前面有g
grep -n [^g]oo regular_express.txt
# 不想要小写字母开头的xoo
grep -n [^a-z]oo regular_express.txt
# 匹配行首the开头的行
grep -n '^the' regular_express.txt
#找出空白行
grep -n '^$' regular_express.txt

扩展grep(grep -E 或者 egrep)：
使用扩展grep的主要好处是增加了额外的正则表达式元字符集。
#找出包含NW或EA的行
egrep 'NW|EA' testfile

不使用正则表达式
fgrep 查询速度比grep命令快，但是不够灵活：它只能找固定的文本，而不是规则表达式。
#如果你想在一个文件或者输出中找到包含星号字符的行
fgrep  '*' /etc/profile
```

#### wc  统计命令

```
wc [-clw][--help][--version][文件...]
-c或--bytes或--chars 只显示Bytes数。
-l或--lines 显示行数。
-w或--words 只显示字数。
--help 在线帮助。
--version 显示版本信息。
```

### 问题

#### **一个日志里面，第一个是日期，第二个是日志级别，第三个是message，需要过滤某一段时间内 error 的数量，命令是啥？**

答：比如统计今天11点到12点的error日志数量：
cat xxx.log | grep "2022-03-15 11:" | grep "error" | wc -l





## 其它

### 统一认证流程

![img](https://cdn.jsdelivr.net/gh/jianhaojiang/PicGoBed/img/20220402112011.png)

1. 用户在第三方应用上点击登录，应用向认证服务器发送请求，说有用户希望进行授权操作，同时说明自己是谁、用户授权完成后的回调url。
2. 认证服务器展示给用户自己的授权界面。
3. 用户进行授权操作，认证服务器验证成功后，生成一个授权编码code，并跳转到第三方的回调url。
4. 第三方应用拿到code后，连同自己在平台上的身份信息（ID密码）发送给认证服务器，再一次进行验证请求，说明自己的身份正确，并且用户也已经授权我了，来换取访问用户资源的权限。
5. 认证服务器对请求信息进行验证，如果没问题，就生成访问资源服务器的令牌access_token，交给第三方应用。
6. 第三方应用使用access_token向资源服务器请求资源。
7. 资源服务器验证access_token成功后返回响应资源。



