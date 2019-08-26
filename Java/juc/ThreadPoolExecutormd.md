## ThreadPoolExecutor

## 方法

`ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler);`



## 参数

`corePoolSize`:	线程池维护线程的最少数量

`maximumPoolSize`:	线程池维护线程的最大数量

`keepAliveTime`:	线程池维护线程所允许的空闲时间

`unit`:	线程池维护线程所允许的空闲时间的单位

`workQueue`:	线程池所使用的缓冲队列

`threadFactory`:	创建新线程时用到的工厂

`handler`:	线程池对拒绝任务的处理策略



## execute(Runnable)方法

执行execute(Runnable)

1. 如果此时线程池中的正在运行的线程数小于 `corePoolSize` ，对 `addWorker(Runnable, boolean)` 的调用需要对 `runState` 和 `workerCount` 做原子校验，通过返回false来防止在不应该添加线程时发生的错误警报。
2. 如果任务可以成功排队，那么还需要仔细检查是否应该添加一个线程（因为自上次检查后现有的线程已经死亡），或者自从进入次方法后池关闭了。所以必须重新检查状态，如果必要的话，则回滚入队，或者如果没有，则启动新的线程。
3. 如果我们不能排队任务，那么我们尝试添加一个新线程。如果失败，我们知道我们已关闭或饱和，因此拒绝该任务。



## 拒绝策略

1. `AbortPolicy`：抛出`RejectedExecutionExeception`异常；
2. `CallRunsPolicy`：用于被拒绝任务的处理程序，它直接在execute方法的调用线程中运行被拒绝的任务；如果执行程序已关闭，则会丢弃该任务；
3. `DiscardOldestPolicy`：丢弃最老的一个请求，也就是即将被执行的任务，并尝试再次提交当前任务；
4. `DiscardPolicy`：丢弃无法处理的任务，不做任何处理；

### 自定义拒绝策略

以上四种拒绝策略都实现了`RejectedExecutionHandler`接口，如果要自定义拒绝策略；可以同这些做法。