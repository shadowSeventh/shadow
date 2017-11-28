---
title: Quartz
date: 2017-11-27 10:33:22
categories: TaskScheduler
tags: [Java,Task]
description: 目前的 Web 应用，多数应用都具备任务调度的功能。本系列大致介绍了几种任务调度的 Java 实现方法，包括 Timer,Scheduler, Quartz 以及 JCron Tab，并对其优缺点进行比较
---

### 先看一个简单的Demo

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
`TestQuartz.main()`依次创建了`scheduler`（调度器）、`job`（任务）、`trigger`（触发器），其中，`job`指定了`JobImpl`，`trigger`保存`job`的触发执行策略（每隔2s执行一次），`scheduler`将`job`和`trigger`绑定在一起，最后`scheduler.start()`启动调度，每隔2s触发执行`JobImpl.execute()`，打印出job impl running。

对于`quartz.properties`，简单场景下，开发者不用自定义配置，使用`quartz`默认配置即可，但在要求较高的使用场景中还是要自定义配置，比如通过`org.quartz.threadPool.threadCount`设置足够的线程数可提高多job场景下的运行性能。更详尽的配置见[官网配置说明页](http://www.quartz-scheduler.org/documentation/quartz-2.1.x/configuration/)。
### job（任务）
job由若干个`class`和`interface`实现。
#### Job接口
开发者想要`job`完成什么样的功能，必须且只能由开发者自己动手来编写实现，比如demo中的`JobImpl`，这点无容置疑。但要想让自己的`job`被`quartz`识别，就必须按照`quartz`的规则来办事，这个规则就是`job`实现类必须实现Job接口，比如`JobImpl`就实现了Job。

Job只有一个`execute(JobExecutionContext)`，`JobExecutionContext`保存了`job`的上下文信息，比如绑定的是哪个`trigger`。`job`实现类必须重写`execute()`，执行job实际上就是运行`execute()`。
#### JobDetailImpl类 / JobDetail接口
`JobDetailImpl`类实现了`JobDetail`接口，用来描述一个job，定义了job所有属性及其get/set方法。了解job拥有哪些属性，就能知道quartz能提供什么样的能力，下面笔者用表格列出job若干核心属性。

|属性名	| 说明|
|--------|-----------------------|
| class | 必须是job实现类（比如`JobImpl`），用来绑定一个具体job|
| name | job名称。如果未指定，会自动分配一个唯一名称。所有job都必须拥有一个唯一name，如果两个job的name重复，则只有最前面的job能被调度|
| group | job所属的组名 |
| description | job描述 |
| durability | 是否持久化。如果job设置为非持久，当没有活跃的trigger与之关联的时候，job会自动从scheduler中删除。也就是说，非持久job的生命期是由trigger的存在与否决定的 |
| shouldRecover | 是否可恢复。如果job设置为可恢复，一旦job执行时scheduler发生hard shutdown（比如进程崩溃或关机），当scheduler重启后，该job会被重新执行|
| jobDataMap | 除了上面常规属性外，用户可以把任意kv数据存入jobDataMap，实现job属性的无限制扩展，执行job时可以使用这些属性数据。此属性的类型是`JobDataMap`，实现了`Serializable`接口，可做跨平台的序列化传输 |

#### JobBuilder类
```java
// 创建任务
JobDetail jobDetail = JobBuilder.newJob(JobImpl.class).withIdentity("myJob", "jobGroup").build();
```
上面代码是demo一个片段，可以看出`JobBuilder`类的作用：接收job实现类`JobImpl`，生成`JobDetail`实例，默认生成`JobDetailImpl`实例。

这里运用了建造者模式：`JobImpl`相当于Product；`JobDetail`相当于Builder，拥有job的各种属性及其get/set方法；`JobBuilder`相当于Director，可为一个job组装各种属性。

### trigger（触发器）
trigger由若干个class和interface实现。
#### SimpleTriggerImpl类 / SimpleTrigger接口 / Trigger接口
`SimpleTriggerImpl`类实现了`SimpleTrigger`接口，`SimpleTrigger`接口继承了`Trigger`接口，它们表示触发器，用来保存触发job的策略，比如每隔几秒触发job。实际上，quartz有两大触发器：`SimpleTrigger`和`CronTrigger`

Trigger诸类保存了trigger所有属性，同job属性一样，了解trigger属性有助于我们了解quartz能提供什么样的能力，下面笔者用表格列出trigger若干核心属性。

|属性名	|属性类型	|说明|
|---------|----------|----------|
|name|所有trigger通用|trigger名称|
|group|所有trigger通用|trigger所属的组名|
|description|所有trigger通用|trigger描述|
|calendarName|所有trigger通用|日历名称，指定使用哪个Calendar类，经常用来从trigger的调度计划中排除某些时间段|
|misfireInstruction|所有trigger通用|错过job（未在指定时间执行的job）的处理策略，默认为`MISFIRE_INSTRUCTION_SMART_POLICY`。|
|priority|所有trigger通用|优先级，默认为5。当多个trigger同时触发job时，线程池可能不够用，此时根据优先级来决定谁先触发|
|jobDataMap|所有trigger通用|同job的jobDataMap。假如job和trigger的jobDataMap有同名key，通过`getMergedJobDataMap()`获取的jobDataMap，将以trigger的为准|
|startTime|所有trigger通用|触发开始时间，默认为当前时间。决定什么时间开始触发job|
|endTime|所有trigger通用|触发结束时间。决定什么时间停止触发job|
|nextFireTime|SimpleTrigger私有|	下一次触发job的时间|
|previousFireTime|SimpleTrigger私有|上一次触发job的时间|
|repeatCount|SimpleTrigger私有|需触发的总次数|
|timesTriggered|SimpleTrigger私有|已经触发过的次数|
|repeatInterval|SimpleTrigger私有|触发间隔时间|

#### TriggerBuilder类
```java
// 创建触发器
// withIntervalInSeconds(2)表示每隔2s执行任务
  Date triggerDate = new Date();
  SimpleScheduleBuilder schedBuilder = SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(2).repeatForever();
  TriggerBuilder<Trigger> triggerBuilder  = TriggerBuilder.newTrigger().withIdentity("myTrigger", "triggerGroup");
  Trigger trigger = triggerBuilder.startAt(triggerDate).withSchedule(schedBuilder).build();
```
上面代码是demo一个片段，可以看出`TriggerBuilder`类的作用：生成Trigger实例，默认生成`SimpleTriggerImpl`实例。同`JobBuilder`一样，这里也运用了建造者模式。
### scheduler（调度器）
scheduler主要由`StdScheduler类`、`Scheduler接口`、`StdSchedulerFactory类`、`SchedulerFactory接口`、`QuartzScheduler类`实现。

```java
// 创建调度器
  SchedulerFactory schedulerFactory = new StdSchedulerFactory();
  Scheduler scheduler = schedulerFactory.getScheduler();
......
// 将任务及其触发器放入调度器
  scheduler.scheduleJob(jobDetail, trigger);
// 调度器开始调度任务
  scheduler.start();
```
上面代码是demo一个片段，可以看出这里运用了工厂模式，通过factory类（`StdSchedulerFactory`）生产出scheduler实例（`StdScheduler`）。scheduler是整个quartz的关键，为此，笔者把demo中用到的scheduler接口的源码加上中文注释做个讲解。

* `StdSchedulerFactory.getScheduler()`源码
```java
public Scheduler getScheduler() throws SchedulerException {
        // 读取quartz配置文件，未指定则顺序遍历各个path下的quartz.properties文件
        // 解析出quartz配置内容和环境变量，存入PropertiesParser对象
        // PropertiesParser组合了Properties（继承Hashtable），定义了一系列对Properties的操作方法，比如getPropertyGroup()批量获取相同前缀的配置。配置内容和环境变量存放在Properties成员变量中
        if (cfg == null) {
            initialize();
        }
        // 获取调度器池，采用了单例模式
        // 其实，调度器池的核心变量就是一个hashmap，每个元素key是scheduler名，value是scheduler实例
        // getInstance()用synchronized防止并发创建
        SchedulerRepository schedRep = SchedulerRepository.getInstance();

        // 从调度器池中取出当前配置所用的调度器
        Scheduler sched = schedRep.lookup(getSchedulerName());

        ......

        // 如果调度器池中没有当前配置的调度器，则实例化一个调度器，主要动作包括：
        // 1）初始化threadPool(线程池)：开发者可以通过org.quartz.threadPool.class配置指定使用哪个线程池类，比如SimpleThreadPool。先class load线程池类，接着动态生成线程池实例bean，然后通过反射，使用setXXX()方法将以org.quartz.threadPool开头的配置内容赋值给bean成员变量；
        // 2）初始化jobStore(任务存储方式)：开发者可以通过org.quartz.jobStore.class配置指定使用哪个任务存储类，比如RAMJobStore。先class load任务存储类，接着动态生成实例bean，然后通过反射，使用setXXX()方法将以org.quartz.jobStore开头的配置内容赋值给bean成员变量；
        // 3）初始化dataSource(数据源)：开发者可以通过org.quartz.dataSource配置指定数据源详情，比如哪个数据库、账号、密码等。jobStore要指定为JDBCJobStore，dataSource才会有效；
        // 4）初始化其他配置：包括SchedulerPlugins、JobListeners、TriggerListeners等；
        // 5）初始化threadExecutor(线程执行器)：默认为DefaultThreadExecutor；
        // 6）创建工作线程：根据配置创建N个工作thread，执行start()启动thread，并将N个thread顺序add进threadPool实例的空闲线程列表availWorkers中；
        // 7）创建调度器线程：创建QuartzSchedulerThread实例，并通过threadExecutor.execute(实例)启动调度器线程；
        // 8）创建调度器：创建StdScheduler实例，将上面所有配置和引用组合进实例中，并将实例存入调度器池中
        sched = instantiate();

        return sched;
}
```
上面有个过程是初始化jobStore，表示使用哪种方式存储scheduler相关数据。quartz有两大jobStore：`RAMJobStore和JDBCJobStore`。`RAMJobStore`把数据存入内存，性能最高，配置也简单，但缺点是系统挂了难以恢复数据。`JDBCJobStore`保存数据到数据库，保证数据的可恢复性，但性能较差且配置复杂。

* `QuartzScheduler.scheduleJob(JobDetail, Trigger)`源码
```java
public Date scheduleJob(JobDetail jobDetail,
            Trigger trigger) throws SchedulerException {
        // 检查调度器是否开启，如果关闭则throw异常到上层
        validateState();
        ......
        // 获取trigger首次触发job的时间，以此时间为起点，每隔一段指定的时间触发job
        Date ft = trig.computeFirstFireTime(cal);

        if (ft == null) {
            throw new SchedulerException(
                    "Based on configured schedule, the given trigger '" + trigger.getKey() + "' will never fire.");
        }

        // 把job和trigger注册进调度器的jobStore
        resources.getJobStore().storeJobAndTrigger(jobDetail, trig);
        // 通知job监听者
        notifySchedulerListenersJobAdded(jobDetail);                
        // 通知调度器线程
        notifySchedulerThread(trigger.getNextFireTime().getTime());
        // 通知trigger监听者
        notifySchedulerListenersSchduled(trigger);

        return ft;
    }
```
* `QuartzScheduler.start()`源码
```java
public void start() throws SchedulerException {
        ......
        // 这句最关键，作用是使调度器线程跳出一个无限循环，开始轮询所有trigger触发job
        // 原理详见“如何采用多线程进行任务调度”
        schedThread.togglePause(false);
        ......
    }
```

