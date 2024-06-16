---
title: CPU打满怎么处理
date: 2024-06-16 21:11:17
tags: CPU打满
categories: 面试
---


1.cpu占用很高的3大类型，9大场景:
--------------------

CPU 飙升是一个常见的问题。

在生产环境中，会出现由代码问题导致CPU占用很高，该如何诊断出是哪行java代码导致? 是大家的一项重要基本功，也是大家面试中的家常骗饭。

如果连CPU 飙升的问题都回答不清楚， 都支支吾吾， 面试就很难通过了

下面，用20多年的技术内功洪荒之力，给大家梳理一下 cpu占用很高的三大类型问题，9大问题场景。

![](./2024/06/16/CPU打满怎么处理/1.png)
### 第1大类型导致CPU100%的问题： 业务类问题

![](./2024/06/16/CPU打满怎么处理/2.png)
#### 1.1 死循环

> while(true)条件

导致 CPU 占用率高的最简单但最具破坏性的编程错误之一就是死循环。

当程序中的循环缺乏正确的退出条件或条件从未满足时，就会出现这种情况，

死循环无休止地运行，消耗过多的处理器时间，导致CPU100%

#### 1.2 死锁

发生死锁后，就会存在忙等待或自旋锁等编程问题，从而导致 繁忙等待问题。

即进程在不释放 CPU 的情况下反复检查条件是否满足，会导致 CPU 占用率居高不下。

这种低效率的资源使用会妨碍 CPU 执行其他任务。

#### 1.3 不必要的代码块

在不需要的地方使用`synchronized`块，会导致线程竞争和上下文切换

解决方案:尽量减少同步块的使用范围

### 第2大类型导致CPU100%的问题：并发类问题

![](./2024/06/16/CPU打满怎么处理/3.png)
#### 1.4 大量计算密集型的任务

比如复杂的数学计算，图像处理，视频编码

计算密集型的任务需要大量的计算能力。在没有足够系统资源的情况下运行这些应用程序，可能会导致 CPU 占用率达到 100%，因为它们试图执行高要求的任务。

解决方案:优化算法，使用更高效的库，或者利用并行计算来分摊

#### 1.5 大量并发线程

多个线程同时运行会导致对 CPU 资源的竞争，尤其是当其中许多线程都是资源密集型进程时。

这会导致所有线程获得的 CPU 时间减少，当每个线程都试图完成自己的任务时，CPU 时间可能会被耗尽。

#### 1.6 大量的上下文切换

创建过多的线程，导致频繁的上下文切换

解决方案:使用线程池来管理线程的数量

### 第3大类导致CPU100%的问题：内存类问题

![](./2024/06/16/CPU打满怎么处理/4.png)
#### 1.7 内存不足

当系统内存不足时，就会将磁盘存储作为虚拟内存使用，而虚拟内存的运行速度要慢得多。

这种**过度的分页和交换**会导致 CPU 占用率居高不下，因为处理器需要花费更多时间来管理内存访问，而不是高效地执行进程。

#### 1.8 频繁GC

创建大量的短生命周期的对象，频繁触发GC

解决方案: 优化代码， 减少对象的创建 ，或者调整JVM的参数来优化

#### 1.9 内存泄漏

程序持续分配内存但不释放，会导致频繁的GC

解决方案:使用内存分析工具VisualVM进行检测和修复

2.CPU100%定位的两大神器:
-----------------

想要定位到具体是哪一行的代码导致， 一般都会使用下面的两大神器

*   通常使用的jvm自带的工具jstack，

*   还有一种就是开源神器arthas，


一般而言，arthas还有其它的功能，所以选择它多一点.

![](./2024/06/16/CPU打满怎么处理/5.png)
在后面的讲解中，

会首先讲解jstack， 使用jstack来解决实际遇到的问题，

然后在使用第二大神器 arthas来解决相同的问题，

大家可以在这两个工具使用过程中选择自己比较顺手的， 这里推荐 arthas， 关于arthas的细致学习，请参考的《arthas 学习圣经》 PDF。

> 此面试题的配套视频，请参见《Java面试宝典核心面试题》第二季。

3 CPU 飙升100%的解决思路与方法论
---------------------

![](./2024/06/16/CPU打满怎么处理/6.png)
4 使用jstack 解决CPU 100%问题实操
-------------------------

使用jstack 解决CPU 100%问题，在方法论上要用到两个命令，

*   top 命令查看TOP N线程，

*   jstack命令查看堆栈信息


![](./2024/06/16/CPU打满怎么处理/7.png)
> 此面试题的配套视频，请参见《Java面试宝典核心面试题》第二季。

### 4.1.jstack命令讲解

命令jstack是java堆栈的跟踪工具，可以打印出程序中所有线程的堆栈信息，包括线程状态，调用栈信息，锁信息等。

jstack可以诊断线程死锁、内存泄漏等问题

命令格式: jstack \[options\] pid

> 常用例子: jstack -l pid，查看线程的堆栈信息

堆栈信息解读:

```


yupengdembp:TestYupeng yupeng$ jstack -l 43953  
2024\-06\-08 10:14:45  
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.191\-b12 mixed mode):  
  
"Attach Listener" #10 daemon prio=9 os\_prio=31 tid=0x00007fb54485a000 nid=0x3503 waiting on condition \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
        - None  
  
"Service Thread" #9 daemon prio=9 os\_prio=31 tid=0x00007fb5430b4000 nid=0x3203 runnable \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
        - None  
  
"C1 CompilerThread2" #8 daemon prio=9 os\_prio=31 tid=0x00007fb54407e800 nid=0x3103 runnable \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
        - None  
  
"C2 CompilerThread1" #7 daemon prio=9 os\_prio=31 tid=0x00007fb54400f800 nid=0x4203 runnable \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
        - None  
  
"C2 CompilerThread0" #6 daemon prio=9 os\_prio=31 tid=0x00007fb54285a000 nid=0x4403 waiting on condition \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
        - None  
  
"Monitor Ctrl-Break" #5 daemon prio=5 os\_prio=31 tid=0x00007fb5430ab000 nid=0x4503 runnable \[0x0000700002427000\]  
   java.lang.Thread.State: RUNNABLE  
        at java.net.SocketInputStream.socketRead0(Native Method)  
        at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)  
        at java.net.SocketInputStream.read(SocketInputStream.java:171)  
        at java.net.SocketInputStream.read(SocketInputStream.java:141)  
        at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)  
        at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)  
        at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)  
        - locked <0x000000079570b9e0\> (a java.io.InputStreamReader)  
        at java.io.InputStreamReader.read(InputStreamReader.java:184)  
        at java.io.BufferedReader.fill(BufferedReader.java:161)  
        at java.io.BufferedReader.readLine(BufferedReader.java:324)  
        - locked <0x000000079570b9e0\> (a java.io.InputStreamReader)  
        at java.io.BufferedReader.readLine(BufferedReader.java:389)  
        at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:47)  
  
   Locked ownable synchronizers:  
        - None  
  
"Signal Dispatcher" #4 daemon prio=9 os\_prio=31 tid=0x00007fb544026000 nid=0x4603 runnable \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
        - None  
  
"Finalizer" #3 daemon prio=8 os\_prio=31 tid=0x00007fb544817000 nid=0x5103 in Object.wait() \[0x0000700002098000\]  
   java.lang.Thread.State: WAITING (on object monitor)  
        at java.lang.Object.wait(Native Method)  
        - waiting on <0x0000000795588ed0\> (a java.lang.ref.ReferenceQueue$Lock)  
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)  
        - locked <0x0000000795588ed0\> (a java.lang.ref.ReferenceQueue$Lock)  
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)  
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)  
  
   Locked ownable synchronizers:  
        - None  
  
"Reference Handler" #2 daemon prio=10 os\_prio=31 tid=0x00007fb54303f800 nid=0x2c03 in Object.wait() \[0x0000700001f95000\]  
   java.lang.Thread.State: WAITING (on object monitor)  
        at java.lang.Object.wait(Native Method)  
        - waiting on <0x0000000795586bf8\> (a java.lang.ref.Reference$Lock)  
        at java.lang.Object.wait(Object.java:502)  
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)  
        - locked <0x0000000795586bf8\> (a java.lang.ref.Reference$Lock)  
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)  
  
   Locked ownable synchronizers:  
        - None  
  
"main" #1 prio=5 os\_prio=31 tid=0x00007fb54280e000 nid=0xe03 waiting on condition \[0x0000700001983000\]  
   java.lang.Thread.State: TIMED\_WAITING (sleeping)  
        at java.lang.Thread.sleep(Native Method)  
        at com.jvm.JVMtest.main(JVMtest.java:6)  
  
   Locked ownable synchronizers:  
        - None  
  
"VM Thread" os\_prio=31 tid=0x00007fb544816800 nid=0x5303 runnable   
  
"GC task thread#0 (ParallelGC)" os\_prio=31 tid=0x00007fb544009800 nid=0x2507 runnable   
  
"GC task thread#1 (ParallelGC)" os\_prio=31 tid=0x00007fb54300f800 nid=0x2403 runnable   
  
"GC task thread#2 (ParallelGC)" os\_prio=31 tid=0x00007fb543010000 nid=0x2303 runnable   
  
"GC task thread#3 (ParallelGC)" os\_prio=31 tid=0x00007fb543010800 nid=0x2a03 runnable   
  
"VM Periodic Task Thread" os\_prio=31 tid=0x00007fb5430b4800 nid=0x3f03 waiting on condition   
  
JNI global references: 15  
  



```

你会发现上面的信息其实是一段一段的，摘取其中的一段为大家说明:

```


"main" #1 prio=5 os\_prio=31 tid=0x00007fb54280e000 nid=0xe03 waiting on condition \[0x0000700001983000\]  
   java.lang.Thread.State: TIMED\_WAITING (sleeping)  
        at java.lang.Thread.sleep(Native Method)  
        at com.jvm.JVMtest.main(JVMtest.java:6)  



```

`main`:线程名称

`#1`:当前线程ID，从main开始，jvm会根据线程创建的顺序为其线程编号

`prio`:优先级的顺序，一般默认是5

`os_prio`:线程对应系统的优先级

`tid`:java内的线程id

`nid`:操作系统级别的线程id，是一个十六进制

关于线程的信息:

`NEW`:线程新建，还没开始运行

`RUNNABLE`:正在java虚拟机中运行的线程

`BLOCKED` :被阻塞，正在等待监视器锁的线程

`WAITING` :无限期等待另一个线程执行特定操作的线程

`TIMED_WAITING`:等待另一个线程执行操作达到指定等待时间的线程

`TERMINATED`:已经退出的线程

我们这里关注的最多的就是`nid`

### 4.2.使用jstack解决CPU占用很高的问题并定位具体行数

先来看一段代码:

```


package com.jvm;  
  
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  
  
public class JVMCPU {  
    private static ExecutorService service = Executors.newFixedThreadPool(5);  
    private static Object lock = new Object();  
    public static class yupengTask implements Runnable{  
  
        @Override  
        public void run() {  
            synchronized (lock){  
                long sum = 0L;  
                while(true){  
                    sum +=1;  
                }  
            }  
        }  
    }  
    public static void main(String\[\] args) {  
  
        yupengTask yupengTask = new yupengTask();  
        service.execute(yupengTask);  
  
    }  
}  
  



```

将这段代码上传到linux服务器，并且使用`nohup java JVMCPU &`运行

![](./2024/06/16/CPU打满怎么处理/8.png)
使用`top`命令可以看到cpu被打满了

![](./2024/06/16/CPU打满怎么处理/9.png)
知道了进程的PID，如何找到进程下是哪个线程呢?可以使用命令`top -Hp 26964`，如下所示

![](./2024/06/16/CPU打满怎么处理/10.png)
从上面的图可以看到，cpu占用最多的线程是26976这个线程id，接下来就是使用`jstack`命令来查看程序的所有堆栈信息，但是，这里需要有一个注意的点，26876这个是一个十进制

的，使用jstack看到的nid是十六进制，所以我们需要转换，可以使用`printf "%x\n"`这个命令

![](./2024/06/16/CPU打满怎么处理/11.png)
接下来使用`jstack -l 26964`打印堆栈信息

```


2024\-06\-08 12:17:36  
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.281\-b09 mixed mode):  
  
"Attach Listener" #10 daemon prio=9 os\_prio=0 tid=0x00007f006c001000 nid=0xc7f waiting on condition \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
	- None  
  
"DestroyJavaVM" #9 prio=5 os\_prio=0 tid=0x00007f00a0009800 nid=0x6955 waiting on condition \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
	- None  
  
"pool-1-thread-1" #8 prio=5 os\_prio=0 tid=0x00007f00a00f0000 nid=0x6960 runnable \[0x00007f008b0ef000\]  
   java.lang.Thread.State: RUNNABLE  
	at JVMCPU$yupengTask.run(JVMCPU.java:14)  
	- locked <0x00000000f59dfcf0\> (a java.lang.Object)  
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)  
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)  
	at java.lang.Thread.run(Thread.java:748)  
  
   Locked ownable synchronizers:  
	- <0x00000000f59e0ed0\> (a java.util.concurrent.ThreadPoolExecutor$Worker)  
  
"Service Thread" #7 daemon prio=9 os\_prio=0 tid=0x00007f00a00d4800 nid=0x695e runnable \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
	- None  
  
"C1 CompilerThread1" #6 daemon prio=9 os\_prio=0 tid=0x00007f00a00b9800 nid=0x695d waiting on condition \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
	- None  
  
"C2 CompilerThread0" #5 daemon prio=9 os\_prio=0 tid=0x00007f00a00b6800 nid=0x695c waiting on condition \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
	- None  
  
"Signal Dispatcher" #4 daemon prio=9 os\_prio=0 tid=0x00007f00a00b5000 nid=0x695b runnable \[0x0000000000000000\]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
	- None  
  
"Finalizer" #3 daemon prio=8 os\_prio=0 tid=0x00007f00a0082000 nid=0x695a in Object.wait() \[0x00007f008b6f5000\]  
   java.lang.Thread.State: WAITING (on object monitor)  
	at java.lang.Object.wait(Native Method)  
	- waiting on <0x00000000f5988ee0\> (a java.lang.ref.ReferenceQueue$Lock)  
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)  
	- locked <0x00000000f5988ee0\> (a java.lang.ref.ReferenceQueue$Lock)  
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)  
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)  
  
   Locked ownable synchronizers:  
	- None  
  
"Reference Handler" #2 daemon prio=10 os\_prio=0 tid=0x00007f00a007d800 nid=0x6959 in Object.wait() \[0x00007f008b7f6000\]  
   java.lang.Thread.State: WAITING (on object monitor)  
	at java.lang.Object.wait(Native Method)  
	- waiting on <0x00000000f5986c00\> (a java.lang.ref.Reference$Lock)  
	at java.lang.Object.wait(Object.java:502)  
	at java.lang.ref.Reference.tryHandlePending(Reference.java:191)  
	- locked <0x00000000f5986c00\> (a java.lang.ref.Reference$Lock)  
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)  
  
   Locked ownable synchronizers:  
	- None  
  
"VM Thread" os\_prio=0 tid=0x00007f00a0074000 nid=0x6958 runnable   
  
"GC task thread#0 (ParallelGC)" os\_prio=0 tid=0x00007f00a001e800 nid=0x6956 runnable   
  
"GC task thread#1 (ParallelGC)" os\_prio=0 tid=0x00007f00a0020800 nid=0x6957 runnable   
  
"VM Periodic Task Thread" os\_prio=0 tid=0x00007f00a00d7800 nid=0x695f waiting on condition   
  
JNI global references: 5  
  
  



```

从上面的信息中，可以看到转换的结果和nid是一致的，

从下面的信息中就可以看到问题其实是出现在`JVMCPU.java`的14行左右，这里给出的是14行，但是实际情况是14行的附近

```


"pool-1-thread-1" #8 prio=5 os\_prio=0 tid=0x00007f00a00f0000 nid=0x6960 runnable \[0x00007f008b0ef000\]  
   java.lang.Thread.State: RUNNABLE  
	at JVMCPU$yupengTask.run(JVMCPU.java:14)  
	- locked <0x00000000f59dfcf0\> (a java.lang.Object)  
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)  
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)  
	at java.lang.Thread.run(Thread.java:748)  



```

结合代码来看一下就很容易问题

![](./2024/06/16/CPU打满怎么处理/12.png)
> 此面试题的配套视频，请参见《Java面试宝典核心面试题》第二季。

5.使用arthas解决CPU占用很高的问题，定位具体代码行
------------------------------

使用arthas解决CPU 100%问题，在方法论上要用到两个命令，

*   dashboard 命令查看TOP N线程，

*   thread 命令查看堆栈信息


![](./2024/06/16/CPU打满怎么处理/13.png)
先来运行`arthas`

![](./2024/06/16/CPU打满怎么处理/14.png)
输入1显示如下

![](./2024/06/16/CPU打满怎么处理/15.png)
输入`dashboard`命令可以看到是哪个线程占用cpu最高

![](./2024/06/16/CPU打满怎么处理/16.png)
接下来输入`thread -n 3`，表示最忙的前3个线程并打印信息

![](./2024/06/16/CPU打满怎么处理/17.png)
从上面的图中可以看到arthas和jstack展示的信息差不多，都定位到了`JVMCPU.java`的14行程序

6.死锁导致CPU占用很高的问题分析
------------------

先来看一段代码:

```


public class DeadlockDemo {  
  
    // 创建两个锁对象  
    private static final Object lock1 = new Object();  
    private static final Object lock2 = new Object();  
  
    public static void main(String\[\] args) {  
  
        // 线程1尝试获取lock1，然后获取lock2  
        Thread thread1 = new Thread(() -> {  
            synchronized (lock1) {  
                System.out.println("Thread 1: Holding lock 1...");  
  
                try { Thread.sleep(100); }   
                catch (InterruptedException e) {}  
  
                System.out.println("Thread 1: Waiting for lock 2...");  
  
                synchronized (lock2) {  
                    System.out.println("Thread 1: Holding lock 1 & 2...");  
                }  
            }  
        });  
  
        // 线程2尝试获取lock2，然后获取lock1  
        Thread thread2 = new Thread(() -> {  
            synchronized (lock2) {  
                System.out.println("Thread 2: Holding lock 2...");  
  
                try { Thread.sleep(100); }   
                catch (InterruptedException e) {}  
  
                System.out.println("Thread 2: Waiting for lock 1...");  
  
                synchronized (lock1) {  
                    System.out.println("Thread 2: Holding lock 2 & 1...");  
                }  
            }  
        });  
  
        thread1.start();  
        thread2.start();  
    }  
}  



```

将上面的代码上传到服务器，使用`nohuo java DeadlockDemo &`运行起来

![](./2024/06/16/CPU打满怎么处理/18.png)
接下来使用arthas进行分析

> 这里选择arthas，不选择jstack是因为arthas更加的方便，它的功能也比jstack丰富

输入thread就可以输出线程的统计信息，其中BLOCKED代表当前阻塞的线程数

![](./2024/06/16/CPU打满怎么处理/19.png)
接下来，输入`thread -b`就可以看到线程具体的情况，在下面的图中已经准确的说明了代码在哪一行

![](./2024/06/16/CPU打满怎么处理/20.png)
> 此面试题的配套视频，请参见《Java面试宝典核心面试题》第二季。

7.小提示
-----

工具的选择建议使用arthas，它还有很多的功能在实际中很有用

大家在面试的时候如果遇到cpu被打满该如何排查这样的问题，不要上来就是使用arthas来定位问题，我们的第一反应永远都是回滚版本，

因为在实际中代码的问题需要分析，不会像举例子这么简单，代码经过分析改动再上线，会浪费很多时间，而有的业务是绝对不允许这么操作的，比如电商，金融的业务，

所以在回答面试官的问题时候一定要先说回滚，在说解决办法.


