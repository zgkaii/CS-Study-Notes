
<!-- MarkdownTOC -->
- [JVM分析调优经验](#jvm分析调优经验)
  - [高分配速率(High Allocation Rate)](#高分配速率high-allocation-rate)
  - [过早提升(Premature Promotion)](#过早提升premature-promotion)
- [JVM 疑难情况问题分析](#jvm-疑难情况问题分析)
- [典型案例](#典型案例)
  - [案例一](#案例一)
  - [案例二](#案例二)
    - [现象描述](#现象描述)
    - [发现问题](#发现问题)
    - [解决问题](#解决问题)
    - [启示](#启示)

<!-- /MarkdownTOC -->


# JVM分析调优经验

## 高分配速率(High Allocation Rate)

分配速率(Allocation rate)表示单位时间内分配的内存量。通常使用 MB/sec作 为单位。上一次垃圾收集之后，与下一次GC开始之前的年轻代使用量，两者的差 值除以时间,就是分配速率。分配速率过高就会严重影响程序的性能，在 JVM 中可能会导致巨大的 GC 开销

* 正常系统：分配速率较低 ~ 回收速率 -> 健康 
* 内存泄漏：分配速率 持续大于 回收速率 -> OOM 
* 性能劣化：分配速率很高 ~ 回收速率 -> 亚健康

这里来看一个GC日志：

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529163527433.png" width="500px"></div>

分析可知，JVM开启了3次Young GC，JVM 启动之后 291ms，共创建了 33,280 KB 的对象。第一 次 Minor GC（小型GC） 完成后，年轻代中还有 5,088 KB 的对象存活。在启动之后 446 ms，年轻代的使用量增加到 38,368 KB， 触发第二次 GC，完成后年轻代的使用量减少到 5,120 KB。 在启动之后 829 ms，年轻代的使用量为 71,680 KB，GC 后变为 5,120 KB。

根据这些信息可以计算出堆内存分配速率：

| Event  | Time  | Young Before | Young After | Allocated during | Allocation rate |
| ------ | ----- | ------------ | ----------- | ---------------- | --------------- |
| 1st GC | 291ms | 33,280 KB    | 5,088 KB    | 33,280 KB        | 114MB/sec       |
| 2st GC | 446ms | 38,368 KB    | 5,120 KB    | 33,280 KB        | 215MB/sec       |
| 3st GC | 829ms | 71,680 KB    | 5,120 KB    | 66,560 KB        | 174MB/sec       |
| Total  | 829ms | N/A          | N/A         | 133,120KB        | 161MB/sec       |

如果分配速率越高，说明Young GC或者Minor GC倍填满的的时间越短，它们发生的频率也就越高，那么GC暂停的频率也就越高。

**如何调优**

如何减小频率呢？答案很简单，增大young区的大小，Eden区也就增大了（默认为Young区大小的80%），还可以`-Xmx`增大整堆的大小来调整。由于new 出来的对象，放在Eden区。考虑蓄水池效应，最终的效果是，影响 Minor GC 的次数和时间，进而影响吞吐量。

## 过早提升(Premature Promotion)

提升速率(promotion rate)用于衡量单位时间内从年轻代提升到老年代的数据量。 一般使用 MB/sec 作为单位, 和分配速率类似。

JVM 会将长时间存活的对象从年轻代提升到老年代。根据分代假设，可能存在一 种情况，老年代中不仅有存活时间长的对象,也可能有存活时间短的对象。这就是过早提升：对象存活时间还不够长的时候就被提升到了老年代。

major GC 不是为频繁回收而设计的，但 major GC 现在也要清理这些生命短暂 的对象，就会导致 GC 暂停时间过长。这会严重影响系统的吞吐量。

和分配速率一样 GC 之前和之后的年轻代使用量以及堆内存 使用量。这样就可以通过差值算出老年代的使用量。 还是以上面案例来计算：

| Event  | Time  | Young Decressed | Total Decressed | Promoted  | Promoted rate |
| ------ | ----- | --------------- | --------------- | --------- | ------------- |
| 1st GC | 291ms | 28,192 KB       | 8,920 KB        | 19,272 KB | 66.2MB/sec    |
| 2st GC | 446ms | 33,248 KB       | 11,400 KB       | 21,848 KB | 140.95MB/sec  |
| 3st GC | 829ms | 66,560 KB       | 30,888 KB       | 35,672 KB | 93.14MB/sec   |
| Total  | 829ms | N/A             | N/A             | 76,792KB  | 92.63MB/sec   |

提升速率也会影响 GC 暂停的频率。但分配速率主要影响 minor GC，而提升速率则影响 major GC 的频率。 有大量的对象提升，自然很快将老年代填满。老年代填充的越快，则 major GC 事件的频率就会越高。

一般来说过早提升的症状表现为以下形式： 

1. 短时间内频繁地执行 full GC 
2. 每次 full GC 后老年代的使用率都很低，在10-20%或以下
3. 提升速率接近于分配速率

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529170808713.png" width="500px"></div>

> 指定 GC 参数`-Xmx24m -XX:NewSize=16m -XX:MaxTenuringThreshold=1` 运行程序可以看到上面 GC 日志。

**如何调优**

解决这类问题，需要让年轻代存放得下暂存的数据，有两种简单的方法。

* 一是**增加年轻代的大小**，设置 JVM 启动参数，类似这样： `-Xmx64m -XX:NewSize=32m`，程序在执行时，Full GC 的次数自然会减少很 多，只会对 minor GC 的持续时间产生影响。

* 二是减**少每次业务处理使用的内存数量**（比如把大对象变成小对象），也能得到类似的结果。

> 至于选用哪个方案，要根据业务需求决定。 但总体目标依然是一致的：让临时数据能够在年轻代存放得下。

# JVM 疑难情况问题分析

> 推荐Arthas 诊断分析工具。

发现问题，我们可以这样来分析：

1、查询业务日志，可以发现这类问题：请求压力大，波峰，遭遇降级，熔断等等， 基础服务、外部 API 依赖 出现故障。 

2、查看系统资源和监控信息： 硬件信息、操作系统平台、系统架构； 

* 排查 CPU 负载、内存不足，磁盘使用量、硬件故障、磁盘分区用满、IO 等待、IO 密集、丢数据、并发竞争等 情况； 
* 排查网络：流量打满，响应超时，无响应，DNS 问题，网络抖动，防火墙问题，物理故障，网络参数调整、超 时、连接数。

3、查看性能指标，包括实时监控、历史数据。可以发现假死，卡顿、响应变慢等现象； 

* 排查数据库， 并发连接数、慢查询、索引、磁盘空间使用量、内存使用量、网络带宽、死锁、TPS、查询数据 量、redo 日志、undo、 binlog 日志、代理、工具 BUG。可以考虑的优化包括： 集群、主备、只读实例、分 片、分区； 可能还需要关注系统中使用的各类中间件。

4、排查系统日志， 比如重启、崩溃、Kill 。 

5、APM，比如发现有些链路请求变慢等等。 

6、排查应用系统

* 排查配置文件: 启动参数配置、Spring 配置、JVM 监控参数、数据库参数、Log 参数、内存问题，比如是否存 在内存泄漏，内存溢出、批处理导致的内存放大等等；
* GC 问题， 确定 GC 算法，GC 总耗时、GC 最大暂停时间、分析 GC 日志和监控指标： 内存分配速度，分代提 升速度，内存使用率等数据，适当时修改内存配置； 
* 排查线程, 理解线程状态、并发线程数，线程 Dump，锁资源、锁等待，死锁； 排查代码， 比如安全漏洞、低效代码、算法优化、存储优化、架构调整、重构、解决业务代码 BUG、第三方 库、XSS、CORS、正则； 
* 单元测试：覆盖率、边界值、Mock测试、集成测试。

7、排除资源竞争、坏邻居效应。

8、疑难问题排查分析手段

* DUMP 线程/内存；
* 抽样分析/调整代码、异步化、削峰填谷。

# 典型案例

## 案例一

**现象描述**：线上虚拟机或者云环境的机器（4G），本身压力不大流量不高，但经常有机器内存溢出OOM崩掉了，架构师就一直加机器，从3台加到15台，进一步分摊了流量，变成几星期崩掉几台台机器，但是还有十台机器不妨碍对外提供的服务。

**发现问题**：JVM参数配置的`-Xmx4g`。这只是限制了 Heap 部分的最大值为 4g（堆内内存），这个内存不包括栈内存，也不包括堆外使用的内存，同时JVM内部也会消耗一定系统内存。算下来其实留给堆内的内存估计就3.2~3.4G左右了。

**解决问题**：JVM参数配置的`-Xmx3g`。机器也顺利减至3台，稳定运行几个月也没有崩掉。

**启示**：（1）尽量不要在线上做Java应用程序的混合部署。也就是说一台服务器，比如说资源特别多核数特别高64核或256核，我们可以做一些逗号化和虚拟化，让线上的每一个Java应用程序都运行在自己的虚拟机中或者Docker中，做资源隔离，这样进程与进程之间就不会抢占资源，相互不会影响；（2）我们`-Xmx`配置的最大值一般指定为整个操作系统的`60%~80%`，不要顶格配置。

## 案例二

我们来看一个实际的案例。 假设我们有一个提供高并发请求的服务，系统使用 Spring Boot框架，指标采集使用 MicroMeter ，监控数据上报给 Datadog 服务。 

> 当然，Micrometer支持将数据上报给各种监控系统， 例如： AppOptics, Atlas, Datadog, Dynatrace, Elastic, Ganglia, Graphite, Humio, Influx, Instana, JMX, KairosDB, New Relic, Prometh eus, SignalFx, Stackdriver, StatsD, Wavefront 等 等。 
>
> 有关MicroMeter的信息可参考: https://micrometer.io/docs

### 现象描述

最近一段时间，通过监控指标发现，有一个服务节点的最大GC暂停时间经常会达到 400ms以上。

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529173212656.png" width="800px"></div>

从图中可以看到，GC暂停时间的峰值达到了 546ms ，这里展示的时间点是 2020年 02月04日09:20:00 左右。 客户表示这种情况必须解决，因为服务调用的超时时间为1秒，要求最大GC暂停时间 不超过200ms，平均暂停时间达到100ms以内，对客户的交易策略产生了极大的影响。

### 发现问题

**CPU的使用情况**如下图所示：

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529173327818.png" width="800px"></div>

从图中可以看到：系统负载为 4.92 , CPU使用率 7% 左右，其实这个图中隐含了一 22.6.2 CPU负载 些重要的线索，但我们此时并没有发现什么问题。

然后排查排查了这段时间的**内存使用情况**：

<div align="center"> <img src="https://img-blog.csdnimg.cn/2021052917343997.png" width="800px"></div>

从图中可以看到，大约 09:25 左右 old_gen 使用量大幅下跌，确实是发生了 FullGC 。 但 09:20 前后，老年代空间的使用量在缓慢上升，并没有下降，也就是说引发最大暂停时间的这个点并没有发生FullGC。 当然，这些是事后复盘分析得出的结论。当时对监控所反馈的信息并不是特别信任， 怀疑就是触发了FullGC导致的长时间GC暂停。

> 为什么有怀疑呢，因为Datadog这个监控系统，默认10秒钟上报一次数据。 有可 能在这10秒内发生些什么事情但是被漏报了（当然，这是不可能的，如果上报失 败会在日志系统中打印相关的错误）

再分析上面这个图，可以看到老年代对应的内存池是 " ps_old_gen "，通过前面的学习，我们知道， ps 代表的是 ParallelGC 垃圾收集器。

查看**JVM的启动参数**，发现是这样的：

```powershell
‐Xmx4g ‐Xms4g
```

我们使用的是JDK8，启动参数中没有指定GC，确定这个服务使用了默认的并行垃圾 收集器。 于是怀疑问题出在这款垃圾收集器上面，因为很多情况下 ParallelGC 为了最大的系统处理能力，即吞吐量，而牺牲掉了单次的暂停时间，导致暂停时间会比较长。

### 解决问题

**使用G1垃圾收集器**：

```powershell
‐Xmx4g ‐Xms4g ‐XX:+UseG1GC ‐XX:MaxGCPauseMillis=50
```

接着服务启动成功，等待健康检测自动切换为新的服务节点，继续查看指标。看看暂停时间，每个节点的GC暂停时间都降下来了，基本上在50ms以内，比较符合我们的预期。

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529173742844.png" width="800px"></div>

过了一段时间，我们发现了个下面这个惊喜（也许是惊吓），如下图所示：

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529173836691.png" width="800px"></div>

运行一段时间后，最大GC暂停时间达到了 1300ms , 情况似乎更恶劣了。继续观察，发现不是个别现象：

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529173927662.png" width="800px"></div>

**注册GC事件监听**:

通过JMX注册GC事件监听，把相关的信息直接打印出来。关键代码：

```java
// 每个内存池都注册监听
for (GarbageCollectorMXBean mbean
	: ManagementFactory.getGarbageCollectorMXBeans()) {
	if (!(mbean instanceof NotificationEmitter)) {
		continue; // 假如不支持监听...
	}
	final NotificationEmitter emitter = (NotificationEmitter) mbean;
    // 添加监听
    final NotificationListener listener = getNewListener(mbean);
    emitter.addNotificationListener(listener, null, null);
}
```

通过这种方式，我们可以在程序中监听GC事件，并将相关信息汇总或者输出到日志。 具体的实现代码在后面的章节【应对容器时代面临的挑战】中给出。 再启动一次，运行一段时间后，看到下面这样的日志信息：

```shell
{
"duration":1869,
"maxPauseMillis":1869,
"promotedBytes":"139MB",
"gcCause":"G1 Evacuation Pause",
"collectionTime":27281,
"gcAction":"end of minor GC",
"afterUsage":
{
 "G1 Old Gen":"1745MB",
 "Code Cache":"53MB",
 "G1 Survivor Space":"254MB",
 "Compressed Class Space":"9MB",
 "Metaspace":"81MB",
 "G1 Eden Space":"0"
},
"gcId":326,
"collectionCount":326,
"gcName":"G1 Young Generation",
"type":"jvm.gc.pause"
}
```

这次实锤了，不是FullGC，而是年轻代GC，而且暂停时间达到了 1869 毫秒。 一点道理都不讲，我认为这种情况不合理，而且观察CPU使用量也不高。 找了一大堆资料，试图证明这个 1869 毫秒 不是暂停时间，而只是GC事件的结束时 间减去开始时间。

**打印GC日志**

既然这些手段不靠谱，那就只有祭出我们的终极手段：打印GC日志。 修改启动参数如下:

```powershell
‐Xmx4g ‐Xms4g ‐XX:+UseG1GC ‐XX:MaxGCPauseMillis=50
‐Xloggc:gc.log ‐XX:+PrintGCDetails ‐XX:+PrintGCDateStamps
```

重新启动，希望这次能排查出问题的原因。运行一段时间，又发现了超长的暂停时间。

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529174406113.png" width="800px"></div>

**分析GC日志**

因为不涉及敏感数据，那么我们把GC日志下载到本地进行分析。 定位到这次暂停时间超长的GC事件，关键的信息如下所示：

```java
Java HotSpot(TM) 64‐Bit Server VM (25.162‐b12) for linux‐amd64 JRE (1.8.0_162
built on Dec 19 2017 21:15:48 by "java_re" with gcc 4.3.0 20080428 (Red Hat
Memory: 4k page, physical 144145548k(58207948k free), swap 0k(0k free)
CommandLine flags:
    ‐XX:InitialHeapSize=4294967296 ‐XX:MaxGCPauseMillis=50 ‐XX:MaxHeapSize=4294967296
    ‐XX:+PrintGC ‐XX:+PrintGCDateStamps ‐XX:+PrintGCDetails ‐XX:+PrintGCTimeStamps
    ‐XX:+UseCompressedClassPointers ‐XX:+UseCompressedOops ‐XX:+UseG1GC
2020‐02‐24T18:02:31.853+0800: 2411.124: [GC pause (G1 Evacuation Pause) (young
	[Parallel Time: 1861.0 ms, GC Workers: 48]
        [GC Worker Start (ms): Min: 2411124.3, Avg: 2411125.4, Max: 2411126.2
        [Ext Root Scanning (ms): Min: 0.0, Avg: 0.3, Max: 2.7, Diff: 2.7, Sum
        [Update RS (ms): Min: 0.0, Avg: 3.6, Max: 6.8, Diff: 6.8, Sum: 172.9
        [Processed Buffers: Min: 0, Avg: 2.3, Max: 8, Diff: 8, Sum: 111]
        [Scan RS (ms): Min: 0.0, Avg: 0.2, Max: 0.5, Diff: 0.5, Sum: 7.7]
        [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.1, Sum
        [Object Copy (ms): Min: 1851.6, Avg: 1854.6, Max: 1857.4, Diff: 5.8,
        [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.6
        [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 48
        [GC Worker Other (ms): Min: 0.0, Avg: 0.3, Max: 0.7, Diff: 0.6, Sum:
        [GC Worker Total (ms): Min: 1858.0, Avg: 1859.0, Max: 1860.3, Diff:
    	[GC Worker End (ms): Min: 2412984.1, Avg: 2412984.4, Max: 2412984.6,	
    [Code Root Fixup: 0.0 ms]
    [Code Root Purge: 0.0 ms]
    [Clear CT: 1.5 ms]
    [Other: 5.8 ms]
    [Choose CSet: 0.0 ms]
        [Ref Proc: 1.7 ms]
        [Ref Enq: 0.0 ms]
        [Redirty Cards: 1.1 ms]
        [Humongous Register: 0.1 ms]
        [Humongous Reclaim: 0.0 ms]
        [Free CSet: 2.3 ms]
    [Eden: 2024.0M(2024.0M)‐>0.0B(2048.0K)
        Survivors: 2048.0K‐>254.0M
        Heap: 3633.6M(4096.0M)‐>1999.3M(4096.0M)]
    [Times: user=1.67 sys=14.00, real=1.87 secs]
```

前后的GC事件都很正常，也没发现FullGC或者并发标记周期，但找到了几个可疑的 点。 

* `physical 144145548k(58207948k free)` JVM启动时，物理内存 137GB, 空闲内存55GB.
* ` [Parallel Time: 1861.0 ms, GC Workers: 48]` 垃圾收集器工作线程 48 个。

我们前面的课程中学习了怎样分析GC日志，一起来回顾一下。

* `user=1.67` 用户线程耗时 1.67秒； 
* `sys=14.00` 系统调用和系统等待时间 14秒； 
* `real=1.87 secs` 实际暂停时间 1.87 秒； 
* GC之前，年轻代使用量2GB，堆内存使用量3.6GB, 存活区2MB，可推断出老年代使用量 1.6GB;
* GC之后，年轻代使用量为0，堆内存使用量2GB，存活区254MB， 那么老年代 大约1.8GB，那么内存提升量为`200MB左右` 。

这样分析之后，可以得出结论：

* 年轻代转移暂停，复制了400MB左右的对象，却消耗了1.8秒，系统调用和系统 等待的时间达到了14秒。
* JVM看到的物理内存137GB
* 推算出JVM看到的CPU内核数量 72个，因为 GC工作线程 72* 5/8 ~= 48 个。

看到这么多的GC工作线程我就开始警惕了，毕竟堆内存才指定了4GB。 按照一般的CPU和内存资源配比，常见的比例差不多是 4核4GB ，4核8GB这样的。

看看对应的CPU负载监控信息：

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529174856324.png" width="800px"></div>

通过和运维同学的沟通，得到这个节点的配置被限制为 4核8GB 。

这样一来，GC暂停时间过长的原因就定位到了:

* **K8S的资源隔离和JVM未协调好，导致JVM看见了72个CPU内核，默认的并行 GC线程设置为: 72* 5/8 + 3 = 48 个 ，但是K8S限制了这个Pod只能使用4个CPU内核的计算量，致使GC发生时，48个线程在4个CPU核心上发生资源竞争，导致大量的上下文切换**。

处置措施为：

* 限制GC的并行线程数量。

事实证明，打印GC日志确实是一个很有用的排查分析方法。

**最终解决**——限制GC的并行线程数量

下面是新的启动参数配置：

```java
‐Xmx4g ‐Xms4g
‐XX:+UseG1GC ‐XX:MaxGCPauseMillis=50 ‐XX:ParallelGCThreads=4
‐Xloggc:gc.log ‐XX:+PrintGCDetails ‐XX:+PrintGCDateStamps
```

这里指定了 `‐XX:ParallelGCThreads=4` , 为什么这么配呢？ 我们看看这个参数的说明。

`‐XX:ParallelGCThreads=n`：设置STW阶段的并行 worker 线程数量。

* 如果逻辑处理器小于等于8个，则默认值 n 等于逻辑处理器的数量。

* 如果逻辑处理器大于8个，则默认值 n 等于处理器数量的 5/8+3 。 在大多数情况下 都是个比较合理的值。 如果是高配置的 SPARC 系统，则默认值 n 大约等于逻辑处 理器数量的 5/16 。

`‐XX:ConcGCThreads=n`：设置并发标记的GC线程数量。

* 默认值大约是 `ParallelGCThreads` 的四分之一。一般来说不用指定并发标记的GC线程数量，只用指定并行的即可。

重新启动之后，看看GC暂停时间指标：

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529175221362.png" width="800px"></div>

红色箭头所指示的点就是重启的时间点， 可以发现， 暂停时间基本上都处于50ms范 围内。 后续的监控发现，这个参数确实解决了问题。

**后续**

经过一段时间的运行，因为数据量爆发，定时任务执行时，每次从数据库加载几百万条数据到内存进行处理... ...

<div align="center"> <img src="https://img-blog.csdnimg.cn/2021052917540363.png" width="800px"></div>

### 启示

通过这个案例，可以看到，JVM问题排查和性能调优主要基于监控数据来进行。 还是那句话：**没有量化，就没有改进** 。

简单汇总一下这里使用到的手段： 

* 指标监控 
* 指定JVM启动内存 
* 指定垃圾收集器 
* 打印和分析GC日志

GC和内存是最常见的JVM调优场景，GC的性能维度主要从下面几个指标评定：

* 延迟，GC中影响延迟的主要因素就是暂停时间。 

* 吞吐量，主要看业务线程消耗的CPU资源百分比，GC占用的部分包括GC暂停时间，以及高负载情况下并发GC消耗的CPU资源。 

* 系统容量，主要说的是硬件配置，以及服务能力。 

只要这些方面的指标都能够满足，各种资源占用也保持在合理范围内，就达成了我们的预期。

