---
title: CountDownLatch 多线程执行同一任务
date: 2022/7/30 20:00:00
tags: 
  - springboot
categories: 
  - Java后端	
  - 多线程
---

# 一、什么是 CountDownlatch

 `CountDownLatch` 是一个同步工具类，它通过一个计数器来实现的,初始值为线程的数量。每当一个线程完成了自己的任务,计数器的值就相应得减1。当计数器到达0时,表示所有的线程都已执行完毕,然后在等待的线程就可以恢复执行任务 



# 二、常用方法详解

- `CountDownLatch(int count)`：`count`为计数器的初始值（一般需要多少个线程执行，`count` 就设置为几）；
- `countDown()`: 每调用一次计数器值 - 1，直到`count`被减为 0，代表所有线程全部执行完毕；
- `getCount()`：获取当前计数器的值；
- `await()`: 等待计数器变为0，即等待所有异步线程执行完毕；
- `boolean await(long timeout, TimeUnit unit)`： 此方法与`await()`区别：
  1. 此方法至多会等待指定的时间，超时后会自动唤醒，若 timeout 小于等于零，则不会等待；
  2. boolean 类型返回值：若指定的等待时间内计数器变为零了，则返回 true；若指定的等待时间过去了，则返回 false。



# 三、CountDownLatch 应用场景

- 某个线程需要**在其他n个线程执行完毕后再向下执行**
2. **多个线程并行执行同一个任务**，提高响应速度

以下为一个多线程并行处理数据后汇总的示例：

```java
private static final Logger LOGGER = LoggerFactory.getLogger(MultiThreadUtil.class);

public List<Integer> multiThreadDeal() {
    // 请求任务列表
    List<Integer> reqlist = new ArrayList<>();
    reqlist.add(1);
    reqlist.add(2);
    // 返回结果列表
    List<Integer> retlist = new ArrayList<>();

    // 初始化线程池
    ExecutorService execpool = Executors.newFixedThreadPool(reqlist.size());
    // 初始化计数器
    CountDownLatch latch = new CountDownLatch(reqlist.size());

    for (int item : reqlist) {
        execpool.submit(() -> {
            try {
                // 依次处理任务，并组装结果
                int data = taskDealFunc(item);
                retlist.add(data);
                LOGGER.info("任务"+item+"已处理完成");
            } catch (Exception e) {
                LOGGER.error(e.getMessage());
            } finally {
                // 线程结束计数器 - 1
                latch.countDown();
            }
        });
    }

    // 阻塞主方法，直到所有线程执行完，线程计数器为0
    try {
        latch.await();
    } catch (InterruptedException e) {
        LOGGER.error(e.getMessage());
    }

    // 关闭线程池
    execpool.shutdown();
    
    //返回汇总结果
    return reqlist;

}

/**
     * 任务处理，可以是sql查询，数值计算等
     */
private int taskDealFunc(int num) {
    // sql查询结果
    return num;
}
```

