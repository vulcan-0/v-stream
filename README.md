# JVM实战

## JVM架构

<img src="https://github.com/vulcan-0/v-stream/blob/master/assets/JVM-Architecture.webp">

## JVM参数

### JVM参数分类

根据JVM参数开头可以区分参数类型，共三类：“-”、“-X”、“-XX”，

**标准参数（-）：所有的JVM实现都必须实现这些参数的功能，而且向后兼容**

> 例子：-verbose:class，-verbose:gc，-verbose:jni……

**非标准参数（-X）：默认JVM实现这些参数的功能，但是并不保证所有JVM实现都满足，且不保证向后兼容**

> 例子：Xms20m，-Xmx20m，-Xmn20m，-Xss128k……

**非Stable参数（-XX）：此类参数各个JVM实现会有所不同，将来可能会随时取消，需要慎重使用**

> 例子：-XX:+PrintGCDetails，-XX:-UseParallelGC，-XX:+PrintGCTimeStamps……

#### 标准参数（-）

| 参数 | 描述 |
| ---- | ---- |
| -server | 指定 JVM 类型为 server 类型 |
| -classpath | 指定 classpath，用 “:” 分隔 |
| -verbose:[class\|gc\|jni] | 输出加载的 class 信息、每次 gc 的信息、使用的 jni 信息 |
| -version | 输出产品版本并退出 |

#### 非标准参数（-X）

| 参数 | 描述 |
| ---- | ---- |
| -Xnoclassgc | 禁用类垃圾收集 |
| -Xloggc:\<file\> | 将 GC 状态记录在文件中（带时间戳） |
| -XshowSettings | 显示所有设置并继续 |
| -XshowSettings:all | 显示所有设置并继续 |
| -XshowSettings:vm | 显示所有与 vm 相关的设置并继续 |
| -XshowSettings:properties | 显示所有属性设置并继续 |
| -XshowSettings:locale | 显示所有与区域设置相关的设置并继续 |

#### 控制内存大小

| 参数 | 等同于 | 使用样例 | 描述 |
| ---- | ---- | ---- | ---- |
| -Xms |  | `-Xms4294967296` `-Xms4000m` `-Xms4G` | Minimum Heap Size，最小堆内存大小/堆内存初始值（-X memory size） |
| -Xmx | -XX:MaxHeapSize | `-Xmx6442450944` `-Xmx6000m` `-Xmx6G` | Maximum Heap Size，最大堆内存大小（-X memory max） |
| -Xmn | -XX:NewSize | `-Xmn1024m` `-Xmn1G` | 新生代内存大小/新生代内存初始值（-X memory new） |
| -XX:MaxNewSize |  | `-XX:MaxNewSize=1G` | 新生代最大内存大小 |
| -XX:PermSize |  | `-XX:PermSize=2G` | 老年代内存大小/老年代内存初始值 |
| -XX:MaxPermSize |  | `-XX:MaxPermSize=3G` | 老年代内存最大值 |
| -XX:MetaspaceSize |  | `-XX:MetaspaceSize=128m` | 元数据空间大小/元数据空间初始值 |
| -XX:MaxMetaspaceSize |  | `-XX:MaxMetaspaceSize=256m` | 元数据空间最大值 |
| -Xss | -XX:ThreadStackSize | `-Xss1024K` | 单个线程的内存大小/虚拟机栈内存大小（-X stack size） |
| -XX:NewRatio |  | `-XX:NewRatio=2` | 老年代与新生代的内存比率，默认是2（新生代占1/3，老年代占2/3） |
| -XX:SurvivorRatio |  | `-XX:SurvivorRatio=8` | Eden 区与 Survivor 区的内存比率，默认是8（Eden 区占8/10，From Survivor 区占1/10，To Survivor 区占1/10） |
| -XX:MinHeapFreeRatio |  | `-XX:MinHeapFreeRatio=40` | 空闲堆内存最小百分比，小于该值则进行扩容，默认是40 |
| -XX:MaxHeapFreeRatio |  | `-XX:MaxHeapFreeRatio=70` | 空闲堆内存最大百分比，大于该值则进行缩容，默认值是70 |

#### 指定使用的垃圾回收器

| 参数 | 描述 |
| ---- | ---- |
| -XX:+UseSerialGC | 使用 Serial 垃圾回收器 |
| -XX:+UseParNewGC | 使用 ParNew 垃圾回收器 |
| -XX:+UseConcMarkSweepGC | 使用 CMS 垃圾回收器 |
| -XX:+UseParallelGC | 使用 Parallel Scavenge 垃圾回收器 |
| -XX:+UseParallelOldGC | 使用 Parallel Old 垃圾回收器 |
| -XX:+UseG1GC | 使用 G1 垃圾回收器 |

#### GC 日志

| 参数 | 描述 |
| ---- | ---- |
| -XX:+PrintGC | 打印 GC 日志 |
| -XX:+PrintGCDetails | 打印 GC 详情 |
| -XX:+PrintGCDateStamps | 打印 GC 时间戳 |
| -XX:+PrintGCTimeStamps | 打印 GC 时间戳 |
| -Xloggc:filename | 将 GC 状态记录在文件中（带时间戳） |
| -XX:+UseGCLogFileRotation | 滚动式存储 GC 日志 |
| -XX:NumberOfGClogFiles=\<number of files\> | GC 日志文件最大数 |
| -XX:GCLogFileSize=\<number\>M (or K) | 单个 GC 日志文件最大大小 |

#### 其他日志

| 参数 | 描述 |
| ---- | ---- |
| -XX:ErrorFile=./hs_err_pid\<pid\>.log | JVM Error 日志文件 |
| -XX:HeapDumpPath=./java_pid\<pid\>.hprof | 内存 dump 文件 |
| -XX:+HeapDumpOnOutOfMemoryError | 发生内存溢出时，dump 内存 |
| -XX:+TraceClassLoading | 输出 class 加载情况 |
| -XX:+TraceClassUnloading | 输出 class 卸载情况 |
| -XX:-TraceLoaderConstraints | 输出常量加载情况 |

## 参考文章

- [Learn JVM Tutorial – Architecture & Working of Java Virtual Machine](https://data-flair.training/blogs/java-virtual-machine-jvm/)
- [JVM (Java Virtual Machine) Architecture](https://www.javatpoint.com/jvm-java-virtual-machine)
- [轻松永远记住经典jvm参数](https://zhuanlan.zhihu.com/p/127352212)
- [Java HotSpot VM Options](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html)
- [JVM Parameters](https://www.javadevjournal.com/java/jvm-parameters/)