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

现象：CPU逐渐飙高，且居高不下

<img src="https://github.com/vulcan-0/v-stream/blob/master/assets/image1001.webp">

