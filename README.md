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

## 参考文章
[Learn JVM Tutorial – Architecture & Working of Java Virtual Machine](https://data-flair.training/blogs/java-virtual-machine-jvm/)
[JVM (Java Virtual Machine) Architecture](https://www.javatpoint.com/jvm-java-virtual-machine)
[轻松永远记住经典jvm参数](https://zhuanlan.zhihu.com/p/127352212)