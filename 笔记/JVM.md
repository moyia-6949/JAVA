# JVM
### 一、class文件

##### 1.IDEA插件

​	BinEd 将class文件二进制内容解析得到**常量池**，**字段**，**属性**，**方法**，**类**，**接口**等信息

​	Jclasslib 解析.class字节码文件，得到底层二进制

##### 2.文件解析

​	CA FE BA BE //标记

​	u2 小版本 u2 大版本

​	u4 常量池数量

​	（u1 常量类型 u(n) 常量描述 ）* 常量池数量

​	u2 类访问标志

​	u2 类索引

​	u2 父类索引

​	u2 该类实现接口数量

​	u2 字段数量

​	（u2 字段类型 u(n) 字段信息描述）* 字段数量

​	u2 方法数量

​	（u8 方法关键字描述 u(n)方法描述）* 方法数量	

​	u2 属性数量

​	u(?)属性信息*属性数量

##### 3.常量池

​		![常量池类型解析](.\picture\6ca1007843b8766cb2b876544adca7e.png)

##### 5.类、字段、方法

类访问标志表

![类访问标记表](.\picture\类访问标志表.png)

***

内部类访问标志表

![方法访问标记表](.\picture\内部类访问标志表.png)

***

字段访问标指表

![方法访问标记表](.\picture\字段访问标志表.png)

***

​	方法访问标志表

![方法访问标记表](.\picture\方法访问标记表.png)

​	四者的类型因为都可由各种关键字组合，故类型处需要对以上的关键字组合进行**按位与**操作，如public volatile static 的字段类型标示为0x0049，后续描述信息需根据常量池内容进行组合最终得出代码

### 二、ClassLoader

##### 1.类型

##### 2.双亲委派

### 三、JMM

##### 1.程序计数器（PC）

##### 2.方法区（MethodArea）

##### 3.本地方发栈（NativeStack）

##### 4.堆（Heap）

##### 5.栈（Stack）

### 四、GC

##### 1.垃圾识别

* 引用计数：有地方引用该对象时计数+1，引用失效-1，计数存在对象头，>0可被认为是存活对象

  存在问题：无法处理循环引用问题，此时需要使用Recyler算法

* 可达性算法：从root出发，遍历所有对象，可被搜索到的可认为是可达对象，在此基础上，还需要不同的垃圾回收器对其进行多次标记后才能确认是否为存活对象

##### 2.垃圾回收器

* **Serial**

  串行回收，Copying算法

* **ParNew**

  并行回收，Copying算法

* **Parallel Scavenge**

  并行回收，Copying算法

* **CMS**

  串并行回收，MarkSweep算法

  分四个阶段：初始标记，并行标记，重复标记，清除垃圾

  * 初始标记：会发生STW，对上一次CMS的清除垃圾阶段产生的浮游垃圾和后续的一些没有被标记的垃圾进行标记

  * 并行标记：在不影响原程序运行的情况下并行标记垃圾
  * 重复标记：串行标记，此时会发生STW。主要针对并行标记期间，标记后又产生引用变更的对象重新标记。比如并行标记期间标记了一个没有被引用的对象，但是过后又有引用指向该对象，那么并行标记的内容就不准确了。
  * 清除垃圾：对前面所标记的垃圾进行清除，如果在此期间产生垃圾则称为浮游垃圾，等待下次CMS标记回收。

  因为使用的是MarkSweep算法，必然会产生碎片化问题，当碎片化到一定程度时会使用SerialOld来兜底整理内存空间

* **SerialOld**

  Mark Compact算法

* **PO**

  并发回收老年代

* **G1**

  算法：三色标记 + SATB

* **ZGC (1ms) PK C++**

  算法：ColoredPointers + LoadBarrier

* **Shenandoah**
  算法：ColoredPointers + WriteBarrier

* **Eplison**

##### 3.垃圾回收器组合

* Serial + SerialOld
* ParNew + CMS
* PS + PO(JVM默认的垃圾回收器组合)

##### 4.分代回收策略

 * 年轻代

   * 组成：Edan+survior1+survior2

   * 年轻代的每个分区大小可通过配置控制。

   * 当对象初始化时先进入Edan，YGC后可回收的被回收，没有被回收的会进入s1，并且年龄+1；再一次YGC会把YGC和s1的对象回收，没有回收的会进入s2，年龄+1；再一次YGC会把Edan和s2回收，没有回收的进入s1，年龄+1；如此循环反复，当达到一定年龄时不会在年轻代流转，会直接进入老年代等待FGC。默认情况下，除CMS设定的年龄上线是6以外，其他的都是15

   * 存在特殊的情况，当对象过大，超过survior的一半时，将直接进入Old

   * 动态年龄：对象按大小从小到大排序，当前面若干个对象的内存占比累加到超过指定值时，下一个对象会直接进入Old

     该阈值可通过 -XX:MaxTenuringThreshold 配置，因为JVM使用4个bit来表示，所以该值最大为15

   * 分配担保：YGC期间 survivor区空间不够了 空间担保直接进入老年代

 * 老年代

   年轻代始终处理不掉的对象存放的内存空间。

##### 5.GC算法



### 五、JVM调优

##### 1.JVM参数

##### 2.性能分析工具

​	图形化分析工具：MAT，jconsole，JMC，jprofiler 

​	命令行工具：

​	step1：top 获取当前java程序pid

​	step2：top -Hp <pid> 观察 该进程中的线程

​	step3：jps 查看运行中的java进程

​	step4：jstack -F <pid>查看该进程下各线程执行情况 （该pid可能需要以十六进制表示）

​	step5：jstat -gc <pid> <mill> 每隔mill毫秒查看该进程GC情况

​	step6：jmap -histo <pid> 查看产生的对象，通常用管道+head/grep获取想要的对象

​	step7：jmap -dump:format=b,file=xxx <pid> 通过该工具可以对对象信息导出分析，具有一定的格式，但是导出的文件偏大，会造成卡顿，影响服务的运行

​	解决：1.设定HeapDump参数，OOM时自动导出堆转储文件

​				2.在高可用的服务器背景下，停掉某台机并不会最总体服务有很大的影响

​				3.在线定位

​	step8：MAT/jhat/jvisualvm可以对dump的文件进行分析

​				jhat是jdk自带的命令，使用jhat -J-mx512M xxx.dump 可以将dump文件解析，并且开启一个服务端口，通过ip:7000访问

​	step9：找到代码问题

##### 3.arthas在线排查工具（gitHub有相关官方使用指引）

java -jar arthas-boot.jar进入

* jvm 观察JVM信息
* thread 定位线程问题
* dashboard 观察系统情况 ，类似jconsole
* jad 反编译
* redefine 热替换，在不停止服务的情况下替换.class文件
* sc -search class
* watch -watch method

##### 4.分析逻辑

四大表象： GC 耗时增大、线程 Block 增多、慢查询增多、CPU 负载高 

通过以下四种判断对根本原因进行分析解刨

* **时序分析：** 先发生的事件是根因的概率更大，通过监控手段分析各个指标的异常时间点，还原事件时间线，如先观察到 CPU 负载高（要有足够的时间 Gap），那么整个问题影响链就可能是：CPU 负载高 -> 慢查询增多 -> GC 耗时增大 -> 线程Block增多 -> RT 上涨。 
* **概率分析：**使用统计概率学，结合历史问题的经验进行推断，由近到远按类型分析，如过往慢查的问题比较多，那么整个问题影响链就可能是：慢查询增多 -> GC 耗时增大 -> CPU 负载高 -> 线程 Block 增多 -> RT上涨。 
*  **实验分析：** 通过故障演练等方式对问题现场进行模拟，触发其中部分条件（一个或多个），观察是否会发生问题，如只触发线程 Block 就会发生问题，那么整个问题影响链就可能是：线程Block增多 -> CPU 负载高 -> 慢查询增多 -> GC 耗时增大 -> RT 上涨。 
*  **反证分析：** 对其中某一表象进行反证分析，即判断表象的发不发生跟结果是否有相关性，例如我们从整个集群的角度观察到某些节点慢查和 CPU 都正常，但也出了问题，那么整个问题影响链就可能是：GC 耗时增大 -> 线程 Block 增多 -> RT 上涨。 

不同的根因，后续的分析方法是完全不同的。如果是 CPU 负载高那可能需要用火焰图看下热点、如果是慢查询增多那可能需要看下 DB 情况、如果是线程 Block 引起那可能需要看下锁竞争的情况，最后如果各个表象证明都没有问题，那可能 GC 确实存在问题，可以继续分析 GC 问题了。 

##### 5.实际案例

* **案例一：过早晋升**

  * **设计背景：**

    因为存在动态年龄机制，如果设计不合理会频繁导致对象进入Old，造成资源浪费。需要巧妙的配置 **-XX:MaxTenuringThreshold** ，阈值**过大**会导致本该晋升的对象一直停留在Survior，直到Survior溢出，而一旦溢出，Eden+Survior将不再依据年龄全部提升到Old；如果阈值**过小**，大量的短期对象会直接到Old，引起频繁的FGC，芬待回收失去意义

  * **设计理念：**

     一般情况下 Old 的大小应当为活跃对象的 2~3 倍左右，考虑到浮动垃圾问题最好在 3 倍左右，剩下的都可以分给 Young 区。

  *  **具体场景：**

    原配置为 Young 1.2G + Old 2.8G，通过观察 CMS GC 的情况找到存活对象大概为 300~400M，于是调整 Old 1.5G 左右，剩下 2.5G 分给 Young 区。仅仅调了一个 Young 区大小参数（`-Xmn`），整个 JVM 一分钟 Young GC 从 26 次降低到了 11 次，单次时间也没有增加，总的 GC 时间从 1100ms 降低到了 500ms，CMS GC 次数也从 40 分钟左右一次降低到了 7 小时 30 分钟一次。

* **案例二：动态扩容引起的空间震荡**

  * **设计背景：**

    初始化堆、方法区等空间时只会初始最小值大小的空间存储信息，每次不够用时再去申请，而此时就会进行一次GC。如果最小值设计得太小，而实际产生的对象又特别多，那就会频繁的扩容，也会频繁的GC。

  * **设计理念：**

    对出现的各种空间大小的最大最小值都设置为相同的值， 如 `-Xms` 和 `-Xmx`，`-XX:MaxNewSize` 和 `-XX:NewSize`，`-XX:MetaSpaceSize` 和 `-XX:MaxMetaSpaceSize` 等。 

  * **具体场景：**

     服务**刚刚启动时 GC 次数较多**，最大空间剩余很多但是依然发生 GC，内存空间呈现出折线上升并最后趋于平缓的趋势。

* **案例三：显式GC是否保留**

  * **设计背景：**

    出现CMS的场景有：空间的扩容缩容，Old区达到阈值，MetaSpace空间不足，Young区晋升失败，大对象担保失败等情况，如果以上场景都不是，那么就是显式地调用了System.gc

    问题分析：

    * **保留System.gc：**

      CMS的GC分为两种：Background和Foreground，前者就是CMS的四个阶段，而第二种使用的是MarkSweepCompact这样的算法，会收集Young、Old和MetaSpace空间的垃圾，因为涉及空间压缩，那么会带来长时间的STW，十分影响执行效率

    * **去掉System.gc：**

      去除的话则是会产生内存泄漏问题，导致一些本该回收的对象没有及时回收，更容易产生OOM

  * **设计理念：**

    在OOM和降低效率这两者之间抉择，很明显应该保留System.gc。

    同时JVM还支持配置参数  `-XX:+ExplicitGCInvokesConcurrent` 和 `-XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses` 参数来将 System.gc 的触发类型从 Foreground 改为 Background，同时 Background 也会做 Reference Processing，这样的话就能大幅降低了 STW 开销 

* **???案例四：MetaSpace 区 OOM**

  * **设计背景：**

    为了避免弹性伸缩带来的额外 GC 消耗，我们会将 `-XX:MetaSpaceSize` 和 `-XX:MaxMetaSpaceSize` 两个值设置为固定的，但是这样也会导致在空间不够的时候无法扩容，然后频繁地触发 GC，最终 OOM。所以关键原因就是 ClassLoader 不停地在内存中 load 了新的 Class ，一般这种问题都发生在动态类加载等情况上。 

  * **设计理念：**

    问题定位：可以dump出内存使用快照，然后通过MAT/jhat/jvisualvm这些工具进行分析

  * **具体场景：**

     JVM 在启动后或者某个时间点开始，**MetaSpace 的已使用大小在持续增长，同时每次 GC 也无法释放，调大 MetaSpace 空间也无法彻底解决**。 

* **？？？案例五：CMS Old GC 频繁**

  * **设计背景：**

  * **设计理念：**

     这种情况比较常见，基本都是一次 Young GC 完成后，负责处理 CMS GC 的一个后台线程 concurrentMarkSweepThread 会不断地轮询，使用 `shouldConcurrentCollect()` 方法做一次检测，判断是否达到了回收条件。如果达到条件，使用 `collect_in_background()` 启动一次 Background 模式 GC。轮询的判断是使用 `sleepBeforeNextCycle()` 方法，间隔周期为 `-XX:CMSWaitDuration` 决定，默认为2s。 

  * **具体场景：**

     Old 区频繁的做 CMS GC，但是每次耗时不是特别长，整体最大 STW 也在可接受范围内，但由于 GC 太频繁导致吞吐下降比较多。 

* **案例六：单次 CMS Old GC 耗时长**

  * **设计背景：**

    CMS主要用于处理Old区垃圾，一般都是采用Background模式，其中第一阶段初始标记和第三阶段重复标记都是串行标记，期间会产生STW。

  * **设计理念：**

    * FinalReference：

      找到内存来源后通过优化代码的方式来解决，如果短时间无法定位可以增加 `-XX:+ParallelRefProcEnabled` 对 Reference 进行并行处理。

    * symbol table：

      观察 MetaSpace 区的历史使用峰值，以及每次 GC 前后的回收情况，一般没有使用动态类加载或者 DSL 处理等，MetaSpace 的使用率上不会有什么变化，这种情况可以通过 `-XX:-CMSClassUnloadingEnabled` 来避免 MetaSpace 的处理，JDK8 会默认开启 CMSClassUnloadingEnabled，这会使得 CMS 在 CMS-Remark 阶段尝试进行类的卸载。

  * **具体场景：**

    CMS GC 单次 STW 最大超过 1000ms，不会频繁发生，如下图所示最长达到了 8000ms。某些场景下会引起“雪崩效应”，这种场景非常危险，我们应该尽量避免出现。 

    * 分析方向： 观察详细 GC 日志，找到出问题时 Final Remark 日志，分析下 Reference 处理和元数据处理 real 耗时是否正常，详细信息需要通过 `-XX:+PrintReferenceGC` 参数开启。**基本在日志里面就能定位到大概是哪个方向出了问题，耗时超过 10% 的就需要关注**。 
    * 根本原因：需要dump出内存使用情况然后用MAT、jht、jvisualvm等工具来分析

* **案例七：内存碎片&收集器退化**

  * **设计背景：**

    * **晋升失败：**

      在YGC时survior放不下只能进入Old，但是Old此时也放不下。这种情况在有老年代垃圾回收器和担保机制存在的情况是很少发生的，但是特殊的是，在特定的情况下Old会被迅速占满（如动态年龄设定年龄太小直接晋升，或Old区已经碎片化无法再放入）

    * **增量收集担保失败**

      分配内存失败后，会判断统计得到的 Young GC 晋升到 Old 的平均大小，以及当前 Young 区已使用的大小也就是最大可能晋升的对象大小，是否大于 Old 区的剩余空间。只要 CMS 的剩余空间比前两者的任意一者大，CMS 就认为晋升还是安全的，反之，则代表不安全，不进行Young GC，直接触发Full GC。

    * **显式GC（System.gc）**

    * **并发模式失败（Concurrent Mode Failure）**

      最后一种情况，也是发生概率较高的一种，在 GC 日志中经常能看到 Concurrent Mode Failure 关键字。这种是由于并发 Background CMS GC 正在执行，同时又有 Young GC 晋升的对象要放入到了 Old 区中，而此时 Old 区空间不足造成的。

    * **相关面试题**

      为什么 CMS GC 正在执行还会导致收集器退化呢？主要是由于 CMS 无法处理浮动垃圾（Floating Garbage）引起的。CMS 的并发清理阶段，Mutator 还在运行，因此不断有新的垃圾产生，而这些垃圾不在这次清理标记的范畴里，无法在本次 GC 被清除掉，这些就是浮动垃圾，除此之外在 Remark 之前那些断开引用脱离了读写屏障控制的对象也算浮动垃圾。所以 Old 区回收的阈值不能太高，否则预留的内存空间很可能不够，从而导致 Concurrent Mode Failure 发生。 

  * **设计理念：**

    - **内存碎片：** 通过配置 `-XX:UseCMSCompactAtFullCollection=true` 来控制 Full GC的过程中是否进行空间的整理（默认开启，注意是Full GC，不是普通CMS GC），以及 `-XX: CMSFullGCsBeforeCompaction=n` 来控制多少次 Full GC 后进行一次压缩。
    - **增量收集：** 降低触发 CMS GC 的阈值，即参数 `-XX:CMSInitiatingOccupancyFraction` 的值，让 CMS GC 尽早执行，以保证有足够的连续空间，也减少 Old 区空间的使用大小，另外需要使用 `-XX:+UseCMSInitiatingOccupancyOnly` 来配合使用，不然 JVM 仅在第一次使用设定值，后续则自动调整。
    - **浮动垃圾：** 视情况控制每次晋升对象的大小，或者缩短每次 CMS GC 的时间，必要时可调节 NewRatio 的值。另外就是使用 `-XX:+CMSScavengeBeforeRemark` 在过程中提前触发一次 Young GC，防止后续晋升过多对象。

  * **具体场景：**

     并发的 CMS GC 算法，退化为 Foreground 单线程串行 GC 模式，STW 时间超长，有时会长达十几秒 

* **场景八：堆外内存 OOM

  * **设计背景：**

    JVM 的堆外内存泄漏，主要有两种的原因：

    - 通过 `UnSafe#allocateMemory`，`ByteBuffer#allocateDirect` 主动申请了堆外内存而没有释放，常见于 NIO、Netty 等相关组件。
    - 代码中有通过 JNI 调用 Native Code 申请的内存没有释放。

  * **设计理念：**

    

    

  * **具体场景：**

     内存使用率不断上升，甚至开始使用 SWAP 内存，同时可能出现 GC 时间飙升，线程被 Block 等现象，**通过 top 命令发现 Java 进程的 RES 甚至超过了 `-Xmx` 的大小**。出现这些现象时，基本可以确定是出现了堆外内存泄漏。 

 * **案例九：JNI引发的GC**

   * **设计背景：**

      由于 Native 代码直接使用了 JVM 堆区的指针，如果这时发生 GC，就会导致数据错误。因此，在发生此类 JNI 调用时，禁止 GC 的发生，同时阻止其他线程进入 JNI 临界区，直到最后一个线程退出临界区时触发一次 GC。 

     

   * **设计理念：**

     - 添加 `-XX+PrintJNIGCStalls` 参数，可以打印出发生 JNI 调用时的线程，进一步分析，找到引发问题的 JNI 调用。
     - JNI 调用需要谨慎，不一定可以提升性能，反而可能造成 GC 问题。
     - 升级 JDK 版本到 14，避免 [JDK-8048556](https://bugs.openjdk.java.net/browse/JDK-8048556) 导致的重复 GC。

     

   * **具体场景：**

      在 GC 日志中，出现 GC Cause 为 GCLocker Initiated GC。 

   