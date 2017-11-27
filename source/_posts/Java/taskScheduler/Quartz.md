---
title: Quartz
date: 2017-11-27 10:33:22
categories: TaskScheduler
tags: [Java,Task]
description: 目前的 Web 应用，多数应用都具备任务调度的功能。本系列大致介绍了几种任务调度的 Java 实现方法，包括 Timer,Scheduler, Quartz 以及 JCron Tab，并对其优缺点进行比较
---

#### demo

```java
public class JobImpl implements Job {
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        System.out.println("job impl running");
    }
}
```
```java
public class TestQuartz {
    public static void main(String[] args) throws SchedulerException, InterruptedException {
        // 创建调度器
        SchedulerFactory schedulerFactory = new StdSchedulerFactory();
        Scheduler scheduler = schedulerFactory.getScheduler();

        // 创建任务
        JobDetail jobDetail = JobBuilder.newJob(JobImpl.class).withIdentity("myJob", "jobGroup").build();

        // 创建触发器
        // withIntervalInSeconds(2)表示每隔2s执行任务
        Date triggerDate = new Date();
        SimpleScheduleBuilder schedBuilder = SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(2).repeatForever();
        TriggerBuilder<Trigger> triggerBuilder  = TriggerBuilder.newTrigger().withIdentity("myTrigger", "triggerGroup");
        Trigger trigger = triggerBuilder.startAt(triggerDate).withSchedule(schedBuilder).build();

        // 将任务及其触发器放入调度器
        scheduler.scheduleJob(jobDetail, trigger);
        // 调度器开始调度任务
        scheduler.start();
    }

}
```
`quartz.properties`
```
#调度器名，默认名是QuartzScheduler
org.quartz.scheduler.instanceName= TestQuartzScheduler

#============================================================================
# Configure ThreadPool   配置线程池
#============================================================================
org.quartz.threadPool.class= org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount= 10
org.quartz.threadPool.threadPriority= 5

#============================================================================
# Configure JobStore  配置任务存储方式
#============================================================================
#相当于扫描频率
org.quartz.jobStore.misfireThreshold= 60000
org.quartz.jobStore.class= org.quartz.simpl.RAMJobStore
```
