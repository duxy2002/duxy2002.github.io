# 使用Spring ThreadPoolTaskExecutor实现多线程任务


* corePoolSize： 线程池维护线程的最少数量
* keepAliveSeconds  线程池维护线程所允许的空闲时间
* maxPoolSize   线程池维护线程的最大数量
* queueCapacity 线程池所使用的缓冲队列
* rejectedExecutionHandler 线程池拒绝处理策略
    Reject策略预定义有四种：
    * ThreadPoolExecutor.AbortPolicy策略，是默认的策略,处理程序遭到拒绝将抛出运行时 RejectedExecutionException。
    * ThreadPoolExecutor.CallerRunsPolicy策略 ,调用者的线程会执行该任务,如果执行器已关闭,则丢弃.
    * ThreadPoolExecutor.DiscardPolicy策略，不能执行的任务将被丢弃.
    * ThreadPoolExecutor.DiscardOldestPolicy策略，如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程）.

当一个任务通过execute(Runnable)方法欲添加到线程池时：

1. 当池子大小小于corePoolSize就新建线程，并处理请求
2. 当池子大小等于corePoolSize，把请求放入workQueue（长度为queueCapacity）中，池子里的空闲线程就去从workQueue中取任务并处理
3. 当workQueue放不下新入的任务时，新建线程入池，并处理请求，如果池子大小撑到了maximumPoolSize就用RejectedExecutionHandler来做拒绝处理
4. 另外，当池子的线程数大于corePoolSize的时候，多余的线程会等待keepAliveTime长的时间，如果无请求可处理就自行销毁
也就是：处理任务的优先级为：核心线程corePoolSize、任务队列workQueue、最大线程 maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。

> *注：
>   1.spring 配置的线城池(threadPoolTaskExecutor)由于是spring创建注入的，在首次使用之后，会一直保持corePoolSize个空闲线程，它只会把多余的空闲线程在keepAliveSeconds 时间之后释放，
>          而且线城池不能调用shutdown()方法，否则再次调用，由于线程池已经关闭，会报错。
>
>   2. threadPoolTaskExecutor也可以在配置文件配置多个线城池，防止多有任务之间竞争，或者由于不同任务使用的线城池大小不同等情况。
>
> MaxPoolSize的设定如果比系统支持的线程数还要大时，会抛出java.lang.OutOfMemoryError: unable to create new native thread 异常。
