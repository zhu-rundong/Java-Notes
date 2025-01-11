# JVM参数

JVM参数主要分为以下三种：**标准参数、非标准参数、不稳定参数**。

## 标准参数

标准参数：**不会随着JVM变化而变化**，以**`-`**开头，如：-version、-jar，可以使用java -help查看

## 非标准参数

非标准参数：**可能会随着JVM变化而变化，变化较小**，以**`-X`**开头，如：-Xmx、-Xms、-Xmn、-Xss，可以使用java –X查看。

### 常见的非标准参数

1. -Xms堆最小值：默认值是总内存/64（且小于1G），默认情况下，当堆中可用内存小于40%（-XX: MinHeapFreeRatio调整）时，堆内存会开始增加，一直增加到-Xmx大小。
2. -Xmx堆最大值：默认值是总内存/64（且小于1G），如果Xms和Xmx都不设置，则两者大小会相同，默认情况下，当堆中可用内存大于70%时，堆内存会开始减少，一直减小到-Xms的大小；
3. -Xmn新生代内存：默认是整堆的1/3，包括Eden区和两个Survivor区的总和，写法如： -Xmn1024，-Xmn1024k，-Xmn1024m，-Xmn1g 。新生代按整堆的比例分配，所以，**此值不建议设置**！
4. -Xss线程栈内存：默认1M，一般来说是不需要改的。
5. 打印GC日志：-Xloggc:file与-verbose:gc功能类似，只是将每次GC事件的相关情况记录到一个文件中。

## 不稳定参数

不稳定参数：**也是非标准的参数，主要用于JVM调优与Debug**, 以**`-XX`**开头，如： -XX:+ UseG1GC 、 -XX:+UseParallelGC、 -XX:+PrintGCDetails

### 不稳定参数分为三类

- 性能参数：用于JVM的性能调优和内存分配控制，如内存大小的设置；
- 行为参数：用于改变JVM的基础行为，如GC的方式和算法的选择；
- 调试参数：用于监控、打印、输出jvm的信息；

### 不稳定参数语法规则

1. 布尔类型参数值：
  -XX:+
  `+`表示启用该选项
  -XX:-
  `-`表示关闭该选项
  示例：-XX:+UseG1GC：表示启用G1垃圾收集器
2. 数字类型参数值：
  -XX:
  `=`给选项设置一个数字类型值，可跟随单位，例如：'m'或'M'表示兆字节;'k'或'K'千字节;'g'或'G'千兆字节。32K与32768是相同大小的。
  示例：-XX:MaxGCPauseMillis=500 ：表示设置GC的最大停顿时间是500ms
3. 字符串类型参数值：
  -XX:
  `=`给选项设置一个字符串类型值，通常用于指定一个文件、路径或一系列命令列表。
  示例：-XX:HeapDumpPath=./dump.core

### 常用的不稳定参数

- -XX:+UseSerialGC        配置串行收集器
- -XX:+UseParallelGC      配置PS并行收集器
- -XX:+UseParallelOldGC   配置PO并行收集器
- -XX:+UseParNewGC        配置ParNew并行收集器
- -XX:+UseConcMarkSweepGC 配置CMS并行收集器
- -XX:+UseG1GC            配置G1并行收集器
- -XX:+PrintGCDetails     配置开启GC日志打印
- -XX:+PrintGCTimeStamps  配置开启打印GC时间戳
- -XX:+PrintGCDateStamps  配置开启打印GC日期
- -XX:+PrintHeapAtGC      配置开启在GC时，打印堆内存信息



