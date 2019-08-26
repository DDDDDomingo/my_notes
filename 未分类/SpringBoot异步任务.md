# SpringBoot异步任务

## 1、Spring的异步配置

### 线程池

实质是java.util.concurrent.Executor

1. SimpleAsyncTaskExecutor：不是真的线程池，这个类不重用线程，每次调用都会创建一个新的线程。
2. SyncTaskExecutor：这个类没有实现异步调用，只是一个同步操作。只适用于不需要多线程的地方
3. ConcuttentTaskExecutor：Executor的适配类，不推荐使用。如果ThreadPoolTaskExecutor不满足要求时，才用考虑使用这个类
4. SimpleThreadPoolTaskExecutor：是Quartz的SimpleThreadPool的类。线程池同时被quartz和非quartz使用，才需要使用此类
5. ThreadPoolTaskExecutor：最常使用，推荐。 其实质是对java.util.concurrent.ThreadPoolExecutor的包装



### 配置

## 使用场景

邮件发送

定时任务

## 代码实现

1. 在main类中开启异步注解 `@EnableAsync`

2. 在要执行的异步方法中开启异步 `@Async

   - 

   