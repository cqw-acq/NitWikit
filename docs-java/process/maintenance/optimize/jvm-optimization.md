---
title: JVM 优化
sidebar_position: 7
---

# JVM 优化

## 参数

:::danger

所有的启动参数都必须被放在 `-jar` 前面否则会导致服务器无法启动

:::

以下是推荐的 JVM 优化参数(至少 Java 17 以上)

**以下启动参数需要自己补全内存分配(-Xms 和 -Xmx) 和 服务端核心名(-jar 服务端名字.jar)**

<details>
  <summary>Mukul1127 Flag (ZGC) Plus - 4 核 8G 以上推荐</summary>

Mukul1127 Flag Plus 是现代的启动参数，使用 ZGC 作为垃圾回收，**注意:Mukul1127 Flag Plus 至少需要 4c8g 以上环境才可以发挥出优势，更低配置请使用 Aikar flag plus**，Mukul1127 Flag Plus 需要 Java 17 以上!

```shell
java -XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+UseFMA -XX:+UseVectorCmov -XX:+UseNewLongLShift -XX:+UseFastStosb -XX:+SegmentedCodeCache -XX:+OptimizeStringConcat -XX:+DoEscapeAnalysis -XX:+OmitStackTraceInFastThrow -XX:+AlwaysActAsServerClassMachine -XX:+AlwaysPreTouch -XX:+DisableExplicitGC -XX:NmethodSweepActivity=1 -XX:ReservedCodeCacheSize=400M -XX:NonNMethodCodeHeapSize=12M -XX:ProfiledCodeHeapSize=194M -XX:NonProfiledCodeHeapSize=194M -XX:-DontCompileHugeMethods -XX:MaxNodeLimit=240000 -XX:NodeLimitFudgeFactor=8000 -XX:+UseVectorCmov -XX:+PerfDisableSharedMem -XX:+UseFastUnorderedTimeStamps -XX:+UseCriticalJavaThreadPriority -XX:ThreadPriorityPolicy=1 -XX:+UseZGC -XX:AllocatePrefetchStyle=1 -XX:-ZProactive
```

如果你使用 Java 21 以上，你可以将`-XX:-ZProactive`换`-XX:+ZGenerational`，Java 22 以上必须切换

</details>

<details>
  <summary>Mukul1127 (ZGC) Flag - 使用 GraalVM 时推荐</summary>

GraalVM Java 17+ 的参数，

```shell
java -XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+UseFMA -XX:+UseVectorCmov -XX:+UseNewLongLShift -XX:+UseFastStosb -XX:+SegmentedCodeCache -XX:+OptimizeStringConcat -XX:+DoEscapeAnalysis -XX:+OmitStackTraceInFastThrow -XX:+AlwaysActAsServerClassMachine -XX:+AlwaysPreTouch -XX:+DisableExplicitGC -XX:AllocatePrefetchStyle=3 -XX:NmethodSweepActivity=1 -XX:ReservedCodeCacheSize=400M -XX:NonNMethodCodeHeapSize=12M -XX:ProfiledCodeHeapSize=194M -XX:NonProfiledCodeHeapSize=194M -XX:-DontCompileHugeMethods -XX:+PerfDisableSharedMem -XX:+UseFastUnorderedTimeStamps -XX:+UseCriticalJavaThreadPriority -XX:+EagerJVMCI -Dgraal.TuneInlinerExploration=1 -XX:+UseZGC -XX:AllocatePrefetchStyle=1 -XX:-ZProactive
```

如果你使用 Java 21 以上，你可以将 `-XX:-ZProactive` 换为 `-XX:+ZGenerational` ，Java 22 以上必须切换

### 以下是额外选项

更激进的内联，在 Graal 中通过 `-Dgraal.BaseTargetSpending=160` （默认为 120）和 OpenJDK 中的其他一些标志。具有较大缓存的 CPU 可能会从中受益。

`-Dgraal.OptWriteMotion=true` 和 `-Dgraal.WriteableCodeCache=true` ，它们看起来不稳定，但在 GraalVM 22.3.0+ 中可能更稳定

</details>

<details>
  <summary>Mukul1127 Flag (G1GC)</summary>

```shell
java -XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+UseFMA -XX:+UseVectorCmov -XX:+UseNewLongLShift -XX:+UseFastStosb -XX:+SegmentedCodeCache -XX:+OptimizeStringConcat -XX:+DoEscapeAnalysis -XX:+OmitStackTraceInFastThrow -XX:+AlwaysActAsServerClassMachine -XX:+AlwaysPreTouch -XX:+DisableExplicitGC -XX:NmethodSweepActivity=1 -XX:ReservedCodeCacheSize=400M -XX:NonNMethodCodeHeapSize=12M -XX:ProfiledCodeHeapSize=194M -XX:NonProfiledCodeHeapSize=194M -XX:-DontCompileHugeMethods -XX:MaxNodeLimit=240000 -XX:NodeLimitFudgeFactor=8000 -XX:+UseVectorCmov -XX:+PerfDisableSharedMem  -XX:+UseCriticalJavaThreadPriority -XX:ThreadPriorityPolicy=1 -XX:AllocatePrefetchStyle=3 -XX:+UseG1GC -XX:MaxGCPauseMillis=37 -XX:+PerfDisableSharedMem -XX:G1HeapRegionSize=16M -XX:G1NewSizePercent=23 -XX:G1ReservePercent=20 -XX:SurvivorRatio=32 -XX:G1MixedGCCountTarget=3 -XX:G1HeapWastePercent=20 -XX:InitiatingHeapOccupancyPercent=10 -XX:G1RSetUpdatingPauseTimePercent=0 -XX:MaxTenuringThreshold=1 -XX:G1SATBBufferEnqueueingThresholdPercent=30 -XX:G1ConcMarkStepDurationMillis=5.0 -XX:GCTimeRatio=99 -XX:G1ConcRefinementServiceIntervalMillis=150 -XX:G1ConcRSHotCardLimit=16
```

</details>

## 通用

### 内存优化

如果服务器内存足够，可以加上此参数 `-XX:-ZUncommit` ，要求 JVM 总是提前把要用的内存申请好，

并且阻止 JVM 把临时空出来的内存还给系统。这会加快短时间有大量内存的需求时的速度。

### 大页支持

注意在 WINDOWS 上使用大页，必须要以管理员启动

当然，在动手前，让我们先试一试是不是系统已经支持了这项功能 ，在控制台执行此命令

```shell
java -Xlog:gc+init -XX:+UseLargePages -Xmx1g -version
```

如果出现了以下字样，那么说明不完全兼容：

```shell
UseLargePages disabled， no large pages configured and available on the system.
```

那么就说明当前系统并不支持大页，不过不要急，可以试一下这一行命令：

```shell
java -Xlog:gc+init -XX:+UseTransparentHugePages -Xmx1g -version
```

如果看到 `Large Page Support: Enabled (Transparent)` ，表示你的系统支持透明大页

但是如果你依然不支持或者想要追求极致性能，可以去百度搜索当前的系统如何开启大页，

这里就不再过多的赘述了。(LargePages 对服务器提升相当巨大,在我的电脑上，它提升了 50%的性能)

如果支持 LargePages ，加上此参数

```shell
-XX:+UseLargePages  -XX:LargePageSizeInBytes=2m -XX:+UseHugeTLBFS
```

如果支持 TransparentHugePages (不要把两个都加上，优先 LargePages)，加上此参数

```shell
-XX:+UseTransparentHugePages -XX:LargePageSizeInBytes=2m -XX:+UseHugeTLBFS
```

:::tip

在某些服务器上，开启大页后，会延长 JVM 的启动时间，时间从十秒到十分钟不等

:::

### SIMD

如果你使用的是 Pufferfish 的分支(Purpur，Leaf，Leaves，Gale)，你可以添加此参数

```shell
--add-modules=jdk.incubator.vector
```

### 下载源加速

默认的 SpigotLibraryLoader 下载源在国内访问很慢，如果你使用的是 Leaf，你可以添加参数使用国内下载源：

```shell
-DLeaf.library-download-repo=https://maven.aliyun.com/repository/public
```

### 中文编码

防止乱码

```shell
-Dfile.encoding=UTF-8
```

### 删除垃圾信息

(仅适合 Leaf 或者 Gale)

```shell
-Dgale.log.warning.root=false -Dgale.log.warning.offline.mode=false
```

### 更快的安全随机数发生

```shell
-Djava.security.egd=file:/dev/urandom
```

### 异步日志

```shell
 -Xlog:async
```

异步日志记录可能会导致日志记录的顺序不确定

### 更长的 KeepAlive 时间

```shell
-Dpaper.playerconnection.keepalive=60
```

如果你的网络不好,可以适当延长 keepalive 时间,打开[alternate-keepalive](go.md#use-alternate-keepalive)

## 参数解释

### ZGC 与 G1GC

废话不说，直接上图

ZGC

![](_images/zgc.png)

G1GC

![](_images/g1gc.png)

显然,单次 GC 时间从 32ms 降到 0.012ms

:::tip

你可能会发现`ZGC Cycle Time`很长,但其实你不用关心,因为没有停顿主线程，你只需要关心`ZGC Pause Time`即可

:::

## JDK

对各种 JDK 构建的性能测试

在经过冰块大佬,llf的测试后,我们得出了在 Java 21 上对各个 JDK 的性能排行

### 性能

按照测试结果高到低排序

#### Azul Platform Primes(Zing)

Azul 开发的一款超高性能的 JDK,测试时,我们发现 Zing **第一次**初始性能并不如 Zulu,GraalVM ,但是在测试机上,大约十分钟后 Azul 性能开始反超 Zulu,GraalVM

**注意:如果你使用 Azul,那就不要使用上述任何优化参数,并且,该JDK仅能用于Linux**

目前有点小小的兼容性问题，模组服就不要尝试了

优化参数我们还在探索

#### GraalVM

新神,实测区块生成比 Zulu 快了 10%(其他指标由于没有Nigger没测),JS 执行性能是 Nashorn 两倍以上

#### Zulu OpenJDK

老牌性能神,比 Oracle JDK 快了 20 - 30%

BellSoft Liberica JDK 和 Adoptium Eclipse Temurin 在 MC 上的性能表现不如前三个

### 内存占用

#### IBM OpenJ9

内存占用最小,但是性能非常不好,适合在低内存环境使用

### 不推荐的

* OpenJDK builds by Oracle
* Oracle Java SE Development Kit
* AdoptOpenJDK
* Amazon Corretto
* Microsoft Build of OpenJDK
* 阿里巴巴 Dragonwell
* SapMachine 
* Red Hat OpenJDK
* ojdkbuild