# Java应用CPU飙高

[原文链接](https://www.modb.pro/db/246094)

样例程序：

```java
public class FullGcProblem {

    private static ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(50,
            new ThreadPoolExecutor.DiscardOldestPolicy());

    public static void main(String[] args) throws InterruptedException {
        executor.setMaximumPoolSize(50);

        for (; ; ) {
            modelFit();
            Thread.sleep(100);
        }
    }

    private static void modelFit() {
        List<CardInfo> taskList = getAllCartInfo();
        taskList.forEach(info -> {
            // do something
            executor.scheduleWithFixedDelay(() -> {
                // do something with info
                info.m();
            }, 2, 3, TimeUnit.SECONDS);
        });
    }

    private static List<CardInfo> getAllCartInfo() {
        List<CardInfo> taskList = new ArrayList<>();

        for (int i = 0; i < 100; i++) {
            CardInfo ci = new CardInfo();
            taskList.add(ci);
        }

        return taskList;
    }

    private static class CardInfo {

        BigDecimal price = new BigDecimal("0.0");
        String name = "张三";
        int age = 5;
        Date birthDate = new Date();

        public void m() {
        }

    }

}
```

执行脚本：

```shell
# 编译
javac org/vulcan/light/vstream/FullGcProblem.java

# 运行
java -Xms200M -Xmx200M -XX:+PrintGC org/vulcan/light/vstream/FullGcProblem
```

现象：CPU逐渐飙高，且居高不下。

<img src="https://github.com/vulcan-0/v-stream/blob/master/assets/image1001.png">

`top -Hp 30400`，可以看到PID为`30402`的线程CPU占比很高，且CPU时间也很长：

<img src="https://github.com/vulcan-0/v-stream/blob/master/assets/image1002.png">

`jstack 30400`查看线程运行情况，将PID`30402`转成十六进制为`76c2`，对照下图看，可以看到是一个JVM线程：

<img src="https://github.com/vulcan-0/v-stream/blob/master/assets/image1003.png">

`jstat -gc 30400 500`查看GC情况，可以看到FullGC非常频繁，Eden区满了，老年代也满了：

<img src="https://github.com/vulcan-0/v-stream/blob/master/assets/image1004.png">

`jmap -dump:format=b,file=heapdump.phrof 30400`将内存快照dump下来：

<img src="https://github.com/vulcan-0/v-stream/blob/master/assets/image1005.png">

使用`jvisualvm`查看内存情况，可以看到存在大量的`org.vulcan.light.vstream.FullGcProblem$$Lambda$2`和`org.vulcan.light.vstream.FullGcProblem$CardInfo`对象，定位到具体的类，结合代码就可查出具体的问题了。

<img src="https://github.com/vulcan-0/v-stream/blob/master/assets/image1005.png">