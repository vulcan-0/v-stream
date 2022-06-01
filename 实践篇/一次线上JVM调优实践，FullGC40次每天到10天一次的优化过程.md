# 一次线上JVM调优实践，FullGC40次/天到10天一次的优化过程

[原文链接](https://heapdump.cn/article/1859160)

## 案例背景

作者通过一个多月的努力，将FullGC从每天40次，降到了每10天一次。

调优前，GC数据如下：

<img src="https://a.perfma.net/img/1858865">

服务器配置：2核4G，共4台服务器。JVM启动参数如下：

```shell
-Xms1000M 
-Xmx1800M 
-Xmn350M 
-Xss300K 
-XX:+DisableExplicitGC 
-XX:SurvivorRatio=4 
-XX:+UseParNewGC 
-XX:+UseConcMarkSweepGC 
-XX:CMSInitiatingOccupancyFraction=70 
-XX:+CMSParallelRemarkEnabled 
-XX:LargePageSizeInBytes=128M 
-XX:+UseFastAccessorMethods 
-XX:+UseCMSInitiatingOccupancyOnly 
-XX:+PrintGCDetails 
-XX:+PrintGCTimeStamps 
-XX:+PrintHeapAtGC
```

- 初始堆内存：1G，最大堆内存：1.8G，新生代内存：350M，线程栈大小：300K。
- 禁止显式执行GC，即不允许通过代码来触发GC，即System.gc()无效（-XX:+DisableExplicitGC）。
- Eden区与Survivor区的比率为4:3:3。
- 新生代使用ParNew垃圾回收器，老年代使用CMS垃圾回收器。
- 每次回收均使用设定的阈值回收（-XX:+UseCMSInitiatingOccupancyOnly，`设定的阈值`参考下一条说明）。
- 当老年代内存比率占到70%时，则进行一次FullGC（-XX:CMSInitiatingOccupancyFraction=70）。
- 开启CMS的并发重新标记（-XX:+CMSParallelRemarkEnabled，CMS回收过程：初始标记->并发标记->重新标记->并发清除）。
- 启用大内存页支持，每页128M（-XX:LargePageSizeInBytes=128M）。[内存分页大小对性能的提升原理](https://blog.51cto.com/u_14286115/3332634)
- get，set方法转成本地代码，加快访问速度（-XX:+UseFastAccessorMethods，JDK9已移除）。

## 第一次优化

- 调大新生代内存，以减少YoungGC，提高吞吐量（GC吞吐量计算公式：用户线程执行时间/(GC执行时间+用户线程执行时间)）。
- 将初始堆内存和最大堆内存大小调为一致，避免内存不足导致的内存大小调整。

```shell
-Xmn350M -> -Xmn800M
-XX:SurvivorRatio=4 -> -XX:SurvivorRatio=8
-Xms1000m -> -Xms1800m
```

<img src="https://a.perfma.net/img/1858890">

运行5天后，YoungGC次数减少了一半以上，时间大概减少了400多秒，但是FullGC平均次数反而增加了将近40次。 因此，第一次优化失败。

## 第二次优化

作者的团队发现某个对象存在1万多个实例，占用了将近20M的内存，经过排查发现，是由匿名内部类导致：

```shell
public void doSmthing(T t){
	redis.addListener(new Listener() {
		public void onTimeout() {
			if (t.success()) {
				// 执行操作
			}
		}
	});
}
```

对于执行失败的操作，回调函数需要等到超时（1分钟）之后才会被释放。于是作者通过日志排查，把失败的事件处理掉了。 

本次优化解决了部分内存泄漏问题，发布后服务器依然会莫名重启。 因此，问题根因依然没有找到。

## 第三次优化

作者在服务不忙的时候dump了服务内存：

<img src="https://a.perfma.net/img/1858914">
<img src="https://a.perfma.net/img/1858945">

发现存在大量的ByteArrayRow对象（4万多），并确定了这些对象是在执行数据库操作的时候产生的。 

同时，运维同事发现在一天的某个时候，入口流量翻了好几倍。

<img src="https://a.perfma.net/img/1858960">

作者排查了业务流量，发现并不会这么大，并且不存在文件上传，也确认了并非流量攻击。

正当作者排查入口流量时，他的同事发现某个SQL查询由于筛选条件不对，一次查询查出了40多万条数据。 

（查询出40多万条数据，ByteArrayRow对象只有4万多？大概是由于dump内存时，只有4万多数据到达了服务器，其他还在传输中。）

优化查询后（修改筛选条件，避免一次查出大量数据），并将JVM参数调整为第一次优化前的参数，服务器运行正常（不再莫名重启），运行了3天只有5次FullGC：

<img src="https://a.perfma.net/img/1858985">

## 第四次优化

解决完内存泄漏问题后，作者通过分析GC log发现，前三次FullGC时，老年代内存占用还不到30%。

一顿调查后发现了：[jvm metaspace导致FGC](https://blog.csdn.net/zjwstz/article/details/77478054) （元空间内存不足，导致FullGC）。

于是，对其中2个服务（prod1、prod2）的参数进行了如下优化，prod3、prod4保持不变：

```shell
# prod1
-Xmn350M -> -Xmn800M
-Xms1000M -> -Xms1800M
-XX:CMSInitiatingOccupancyFraction=70 -> -XX:CMSInitiatingOccupancyFraction=75
-XX:MetaspaceSize=200M
```

```shell
# prod2
-Xmn350M -> -Xmn600M
-Xms1000M -> -Xms1800M
-XX:CMSInitiatingOccupancyFraction=70 -> -XX:CMSInitiatingOccupancyFraction=75
-XX:MetaspaceSize=200M
```

线上运行10天左右：

prod1
<img src="https://a.perfma.net/img/1858997">

prod2
<img src="https://a.perfma.net/img/1859024">

prod3
<img src="https://a.perfma.net/img/1859050">

prod4
<img src="https://a.perfma.net/img/1859080">

通过对比发现，prod1、prod2的YoungGC和FullGC次数和时间都明显小于prod3、prod4，说明本次优化效果明显。

而，prod1在10天中只发生了一次FullGC：

<img src="https://a.perfma.net/img/1859091">

<img src="https://a.perfma.net/img/1859101">

此次FullGC的原因为何，作者也尚未调查出来。

作者最后的总结：

- FullGC一天超过一次就不正常了。
- 发现FullGC频繁，应该优先调查内存泄漏问题。
- 内存泄漏解决后，JVM可以调优的空间就比较少了，作为学习还可以，否则不要投入太多的时间。
- 如果发现CPU持续偏高，排除代码问题后可以找运维咨询下阿里云客服，这次调查过程中就发现CPU 100%是由于服务器问题导致的，进行服务器迁移后就正常了。
- 数据查询的时候也是算作服务器的入口流量的，如果访问业务没有这么大量，而且没有攻击的问题的话，可以往数据库方面调查。
- 有必要时常关注服务器的GC，可以及早发现问题。