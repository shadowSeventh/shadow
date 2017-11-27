---
title: Scheduler
date: 2017-11-27 10:33:22
categories: TaskScheduler
tags: [Java,Task]
description: 目前的 Web 应用，多数应用都具备任务调度的功能。本系列大致介绍了几种任务调度的 Java 实现方法，包括 Timer,Scheduler, Quartz 以及 JCron Tab，并对其优缺点进行比较
---

## ScheduledExecutor
### jdk 所实现的 taskScheduler
ScheduledExecutor。设计思想是，每一个被调度的任务都会由线程池中一个线程去执行，因此任务是并发执行的，相互之间不会受到干扰。需 要注意的是，只有当任务的执行时间到来时，ScheduedExecutor 才会真正启动一个线程，其余时间 ScheduledExecutor 都是在轮询任务的状态。

```java
public class ScheduledExecutorTest implements Runnable{
    private String jobName = "";

    public ScheduledExecutorTest(String jobName) {
        super();
        this.jobName = jobName;
    }

    @Override
    public void run() {
        System.out.println("execute " + jobName);
    }
}
```
```java
public class MainExecute {
    public static void main(String[] args) {
        ScheduledExecutorService service = Executors.newScheduledThreadPool(10);

        long initialDelay1 = 1;
        long period1 = 1;
        // 从现在开始1秒钟之后，每隔1秒钟执行一次job1
        service.scheduleAtFixedRate(
                new ScheduledExecutorTest("job1"), initialDelay1,
                period1, TimeUnit.SECONDS);

        long initialDelay2 = 1;
        long delay2 = 1;
        // 从现在开始2秒钟之后，每隔2秒钟执行一次job2
        service.scheduleWithFixedDelay(
                new ScheduledExecutorTest("job2"), initialDelay2,
                delay2, TimeUnit.SECONDS);
    }
}
```
### Spring 所实现的 taskScheduler
#### 核心类： 
* `ScheduledAnnotationBeanPostProcessor`
核心方法：

`postProcessAfterInitialization`负责@Schedule注解的扫描，构建ScheduleTask

`onApplicationEvent`spring容器加载完毕之后调用，ScheduleTask向ScheduledTaskRegistrar中注册, 调用ScheduledTaskRegistrar.afterPropertiesSet() 
* `ScheduledTaskRegistrar`
核心方法：

`afterPropertiesSet`初始化所有定时器，启动定时器
* `TaskScheduler`
主要的实现类有三个`ThreadPoolTaskScheduler`,`ConcurrentTaskScheduler`,`TimerManagerTaskScheduler` 

作用：这些类的作用主要是将`task`和`executor`用`ReschedulingRunnable`包装起来进行生命周期管理。 
核心方法：

`ScheduledFuture schedule`
* `ReschedulingRunnable`
核心方法：`schedule()`,`run()`

使用demo
```java
@Configuration
public class TaskConf {
    @Bean
    public ThreadPoolTaskScheduler initThreadPool() {
        ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
        taskScheduler.setPoolSize(20);
        taskScheduler.initialize();
        return taskScheduler;
    }
}
```
```java
public class TaskService implements Runnable {

    protected Logger log = LoggerFactory.getLogger(getClass());

    @Autowired
    private QhShopProperties qhShopProperties;

    @Autowired
    private OrderRepo orderRepo;

    @Autowired
    private LockRegistry lockRegistry;

    @Autowired()
    @Qualifier("initThreadPool")
    private ThreadPoolTaskScheduler threadPoolTaskScheduler;

    private String jobName;

    public void setJobName(String jobName) {
        this.jobName = jobName;
    }

    public String getLockKey(String key) {
        StringBuilder buf = new StringBuilder();
        buf.append(TaskService.class.getName())
           .append("/").append(key).append("111");
        return buf.toString();
    }


    public void start(Date date) {
        threadPoolTaskScheduler.schedule(this, date);
    }


    @Override
    public void run() {
        String lockKey = getLockKey(jobName);
        long waitLockTime = 1000;
        Lock lock = lockRegistry.obtain(lockKey);
        try {
            if (!lock.tryLock(waitLockTime, TimeUnit.MILLISECONDS)) {
                log.info("加锁失败，任务中止");
                return;
            }
        } catch (InterruptedException e) {
            log.warn("在等待加锁时被中止", e);
            return;
        }
        try {
            System.out.println("execute " + jobName);
        } catch (Exception e) {
        } finally {
            lock.unlock();
        }
    }
}
```
