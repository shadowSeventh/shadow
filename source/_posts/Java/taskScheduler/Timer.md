---
title: TimerTask
date: 2017-11-27 10:33:22
categories: Java
tags: [Java,Task]
description: 目前的 Web 应用，多数应用都具备任务调度的功能。本系列大致介绍了几种任务调度的 Java 实现方法，包括 Timer,Scheduler, Quartz 以及 JCron Tab，并对其优缺点进行比较
---

## TimerTask
### 简单使用
TimerTask 是最简单的任务调度方式之一
```java
 public class TimerTaskTest extends TimerTask {
    private String jobName = "";

    public TimerTaskTest(String jobName) {
        super();
        this.jobName = jobName;
    }

    @Override
    public void run() {
        System.out.println("execute " + jobName);
    }

}
```
调用执行任务
```java
public class MainExecute {
    public static void main(String[] args) {
        Timer timer = new Timer();
        long delay1 = 1 * 1000;
        long period1 = 1000;
        // 从现在开始 1 秒钟之后，每隔 1 秒钟执行一次 job1
        timer.schedule(new TimerTaskTest("job1"), delay1, period1);
        long delay2 = 2 * 1000;
        long period2 = 2000;
        // 从现在开始 2 秒钟之后，每隔 2 秒钟执行一次 job2
        timer.schedule(new TimerTaskTest("job2"), delay2, period2);
    }
}
```
使用 Timer 实现任务调度的核心类是 Timer 和 TimerTask。其中 Timer 负责设定 TimerTask 的起始与间隔执行时间。使用者只需要创建一个 TimerTask 的继承类，实现自己的 run 方法，然后将其丢给 Timer 去执行即可。

Timer 的设计核心是一个 TaskList 和一个 TaskThread。Timer 将接收到的任务丢到自己的 TaskList 中，TaskList 按照 Task 的最初执行时间进行排序。TimerThread 在创建 Timer 时会启动成为一个守护线程`mainLoop`。这个线程会轮询所有任务，找到一个最近要执行的任务，然后休眠，当到达最近要执行任务的开始时间点，TimerThread 被唤醒并执行该任务。之后 TimerThread 更新最近一个要执行的任务，继续休眠。可以看看这个守护线程的源码：
```java
private void mainLoop() {
        while (true) {
            try {
                TimerTask task;
                boolean taskFired;
                synchronized(queue) {
                    // Wait for queue to become non-empty
                    while (queue.isEmpty() && newTasksMayBeScheduled)
                        queue.wait();
                    if (queue.isEmpty())
                        break; // Queue is empty and will forever remain; die

                    // Queue nonempty; look at first evt and do the right thing
                    long currentTime, executionTime;
                    task = queue.getMin();
                    synchronized(task.lock) {   //加锁
                        if (task.state == TimerTask.CANCELLED) {
                            queue.removeMin();
                            continue;  // No action required, poll queue again
                        }
                        currentTime = System.currentTimeMillis();
                        executionTime = task.nextExecutionTime;
                        if (taskFired = (executionTime<=currentTime)) {
                            if (task.period == 0) { // Non-repeating, remove
                                queue.removeMin();
                                task.state = TimerTask.EXECUTED;
                            } else { // Repeating task, reschedule
                                queue.rescheduleMin(
                                  task.period<0 ? currentTime   - task.period
                                                : executionTime + task.period);
                            }
                        }
                    }
                    if (!taskFired) // Task hasn't yet fired; wait
                        queue.wait(executionTime - currentTime);
                }
                if (taskFired)  // Task fired; run it, holding no locks
                    task.run();
            } catch(InterruptedException e) {
            }
        }
    }
```
Timer 的优点在于简单易用，但由于所有任务都是由同一个线程来调度，因此所有任务都是串行执行的，同一时间只能有一个任务在执行，前一个任务的延迟或异常都将会影响到之后的任务。而且，Timer是线程安全的。

### 终止Timer线程
只要一个程序的timer线程在运行，那么这个程序就会保持运行。当然，你可以通过以下四种方法终止一个timer线程：
* 调用timer的cancle方法。你可以从程序的任何地方调用此方法，甚至在一个timer task的run方法里。
* 让timer线程成为一个daemon线程（可以在创建timer时使用new Timer(true)达到这个目地），这样当程序只有daemon线程的时候，它就会自动终止运行。
* 当timer相关的所有task执行完毕以后，删除所有此timer对象的引用（置成null），这样timer线程也会终止。
* 调用System.exit方法，使整个程序（所有线程）终止。

### 更进一步
`Timer` 提供了四种执行方式
* `schedule(TimerTask task, long delay, long period)`
* `schedule(TimerTask task, Date time, long period)`
* `scheduleAtFixedRate(TimerTask task, long delay, long period)`
* `scheduleAtFixedRate(TimerTask task, Date firstTime, long period)`
当计划反复执行的任务时，如果你注重任务执行的平滑度，那么请使用`schedule`方法，如果你在乎的是任务的执行频度那么使用`scheduleAtFixedRate`方法。
`schedule`更注重保持间隔时间的稳定。
`scheduleAtFixedRate`这个方法更注重保持执行频率的稳定。

