# Java 多线程 : JDK 工具

[toc]



## **一、线程池**

### 1. 为什么要使用线程池

使用线程池主要有以下三个原因：

1. 创建/销毁线程需要消耗系统资源，线程池可以**复用已创建的线程**。
2. **控制并发的数量**。并发数量过多，可能会导致资源消耗过多，从而造成服务器崩溃。（主要原因）
3. **可以对线程做统一管理**。



### 2. 线程池原理

Java 中的线程池顶层接口是 `Executor` 接口，`ThreadPoolExecutor` 是这个接口的实现类。

我们可以通过 `ThreadPoolExecutor` 类来创建一个线程池。

#### 2.1 ThreadPoolExecutor 构造方法及参数

共有四个构造方法：

```java
// 五个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue)

// 六个参数的构造函数-1
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory)

// 六个参数的构造函数-2
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          RejectedExecutionHandler handler)

// 七个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

涉及到 5~7 个参数，先看看必须的 5 个参数是什么意思：

- **int corePoolSize**：该线程池中**核心线程数最大值**。

  > 核心线程：线程池中有两类线程，核心线程和非核心线程。核心线程默认情况下会一直存在于线程池中，即使这个核心线程什么都不干（铁饭碗），而非核心线程如果长时间的闲置，就会被销毁（临时工）。

- **int maximumPoolSize**：该线程池中**线程总数最大值** 。

  > 该值等于「核心线程数量 + 非核心线程数量」。

- **long keepAliveTime**：**非核心线程闲置超时时长**。

  > 非核心线程如果处于闲置状态超过该值，就会被销毁。如果设置 `allowCoreThreadTimeOut(true)`，则会也作用于核心线程。

- **TimeUnit unit**：keepAliveTime 的单位。

   ```
  TimeUnit 是一个枚举类型，包括以下属性:
   1.NANOSECONDS: 1微毫秒 = 1微秒 / 1000 
  2.MICROSECONDS: 1微秒 = 1毫秒 / 1000 
   3.MILLISECONDS: 1毫秒 = 1秒 /1000 
  4.SECONDS: 秒 
   5.MINUTES: 分 
  6.HOURS: 小时 
   7.DAYS: 天
  ```
   
- **BlockingQueue workQueue**：阻塞队列，维护着**等待执行的 Runnable 任务对象**。

  常用的几个[阻塞队列](##二、阻塞队列)：

  1. **LinkedBlockingQueue**

     链式阻塞队列，底层数据结构是链表，默认大小是 `Integer.MAX_VALUE`，也可以指定大小。

  2. **ArrayBlockingQueue**

     数组阻塞队列，底层数据结构是数组，需要指定队列的大小。

  3. **SynchronousQueue**

     一个**不存储元素**的同步队列，内部容量为 0，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，反之亦然。

  4. **DelayQueue**

     延迟队列，该队列中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素 。

除了 5 个必须的参数之外，还有两个非必须的参数：

- **ThreadFactory threadFactory**：线程工厂。

  > 创建线程的工厂 ，用于**批量创建线程，统一在创建线程时设置一些参数**，如是否守护线程、线程的优先级等。如果不指定，会新建一个默认的线程工厂。

```java
static class DefaultThreadFactory implements ThreadFactory {
    // 省略属性
    // 构造函数
    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
        Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
            poolNumber.getAndIncrement() +
            "-thread-";
    }
    // 省略
}
```

- **RejectedExecutionHandler handler**：**拒绝处理策略（饱和策略）**。

  线程数量大于最大线程数就会采用拒绝处理策略，四种拒绝处理的策略为 ：

  1. **ThreadPoolExecutor.AbortPolicy**：**默认拒绝处理策略**，丢弃任务并抛出 `RejectedExecutionException` 异常。
  2. **ThreadPoolExecutor.DiscardPolicy**：丢弃新来的任务，但是不抛出异常。
  3. **ThreadPoolExecutor.DiscardOldestPolicy**：丢弃队列头部（最旧的）的任务，然后重新尝试执行程序（如果再次失败，重复此过程）。
  4. **ThreadPoolExecutor.CallerRunsPolicy**：由调用者所在线程处理该任务。



#### 2.2 ThreadPoolExecutor 的策略

线程池本身有一个**调度线程**，这个线程就是**用于管理布控整个线程池里的各种任务和事务**，例如创建线程、销毁线程、任务队列管理、线程队列管理等等。

故线程池也有自己的状态。`ThreadPoolExecutor` 类中使用了一些 `final int` 常量变量来表示线程池的状态 ，分别为 RUNNING、SHUTDOWN、STOP、TIDYING 、TERMINATED。

```java
// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```

- 线程池创建后处于 **RUNNING** 状态。

- 调用 `shutdown()` 方法后处于 **SHUTDOWN** 状态，线程池不能接受新的任务，清除一些空闲 worker，不会等待阻塞队列的任务完成。

- 调用 `shutdownNow()` 方法后处于 **STOP** 状态，线程池不能接受新的任务，中断所有线程，阻塞队列中没有被执行的任务全部丢弃。此时，`poolsize = 0`，阻塞队列的 size 也为0。

- 当所有的任务已终止，`ctl` 记录的”任务数量”为 0，线程池会变为 **TIDYING** 状态。接着会执行 `terminated()` 函数。

  > ThreadPoolExecutor 中有一个控制状态的属性叫 `ctl`，它是一个 AtomicInteger 类型的变量。线程池状态就是通过 AtomicInteger 类型的成员变量 `ctl` 来获取的。
  >
  > 获取的 `ctl` 值传入 `runStateOf` 方法，与 `~CAPACITY` 位与运算（`CAPACITY` 是低 29 位全 1 的 int 变量）。
  >
  > `~CAPACITY` 在这里相当于掩码，用来获取 `ctl` 的高 3 位，表示线程池状态；而另外的低 29 位用于表示工作线程数。

- 线程池处在 TIDYING 状态时，**执行完 `terminated()` 方法之后**，就会由 **TIDYING → TERMINATED**， 线程池被设置为 TERMINATED 状态。



#### 2.3 线程池主要的任务处理流程

处理任务的核心方法是 `execute`，看看 JDK 1.8 源码中 `ThreadPoolExecutor` 是如何处理线程任务的：

```java
// JDK 1.8 
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();   
    int c = ctl.get();
    // 1.当前线程数小于corePoolSize,则调用addWorker创建核心线程执行任务
    if (workerCountOf(c) < corePoolSize) {
       if (addWorker(command, true))
           return;
       c = ctl.get();
    }
    // 2.如果不小于corePoolSize，则将任务添加到workQueue队列。
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 2.1 如果isRunning返回false(状态检查)，则remove这个任务，然后执行拒绝策略。
        if (! isRunning(recheck) && remove(command))
            reject(command);
            // 2.2 线程池处于running状态，但是没有线程，则创建线程
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 3.如果放入workQueue失败，则创建非核心线程执行任务，
    // 如果这时创建非核心线程失败(当前线程总数不小于maximumPoolSize时)，就会执行拒绝策略。
    else if (!addWorker(command, false))
         reject(command);
}
```

`ctl.get()` 是获取线程池状态，用 `int` 类型表示。第二步中，入队前进行了一次 `isRunning` 判断，入队之后，又进行了一次 `isRunning` 判断。

**为什么要二次检查线程池的状态?**

在多线程的环境下，线程池的状态是时刻发生变化的。很有可能刚获取线程池状态后线程池状态就改变了。判断是否将 `command` 加入 `workqueue` 是线程池之前的状态，倘若没有二次检查，万一线程池处于非 **RUNNING** 状态（在多线程环境下很有可能发生），那么 `command` 永远不会执行。

**总结一下处理流程：**

1. 线程总数量 < corePoolSize，无论线程是否空闲，都会新建一个核心线程执行任务（让核心线程数量 < corePoolSize 时快速达到 corePoolSize）。**注意，这一步需要获得全局锁**。
2. 线程总数量 >= corePoolSize 时，新来的线程任务会进入任务队列中等待，然后空闲的核心线程会依次去缓存队列中取任务来执行（体现了**线程复用**）。
3. 当缓存队列满了，说明这个时候任务已经多到爆棚，需要一些“临时工”来执行这些任务了。于是会创建非核心线程去执行这个任务。**注意，这一步需要获得全局锁**。
4. 缓存队列满了，且总线程数达到了 maximumPoolSize，则会采取上面提到的拒绝策略进行处理。

整个过程如图所示：

![img](./imgs/1638705643444.png)



#### 2.4 ThreadPoolExecutor 如何做到线程复用？

我们知道，一个线程在创建的时候会指定一个线程任务，当执行完这个线程任务之后，线程自动销毁。但是线程池却可以复用线程，即**一个线程执行完线程任务后不销毁，继续执行另外的线程任务**。那么，线程池如何做到线程复用呢？

原来，ThreadPoolExecutor 在创建线程时，会将线程封装成**工作线程 worker**，并放入**工作线程组**中，然后**这个 worker 会反复从阻塞队列中拿任务去执行**。下面继续看看源码（一定要仔细看，前后有联系）

这里的 `addWorker` 方法是在上面提到的 `execute` 方法里面调用的，先看看上半部分：

```java
// ThreadPoolExecutor.addWorker方法源码上半部分
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                // 1.如果core是ture,证明需要创建的线程为核心线程，则先判断当前线程是否大于核心线程
                // 如果core是false,证明需要创建的是非核心线程，则先判断当前线程数是否大于总线程数
                // 如果不小于，则返回false
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }
```

上半部分主要是判断线程数量是否超出阈值，超过了就返回 false。继续看下半部分:

```java
    // ThreadPoolExecutor.addWorker方法源码下半部分
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        // 1.创建一个worker对象
        w = new Worker(firstTask);
        // 2.实例化一个Thread对象
        final Thread t = w.thread;
        if (t != null) {
            // 3.线程池全局锁
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                // 4.启动这个线程
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

创建 `worker` 对象，并初始化一个 `Thread` 对象，然后启动这个线程对象。

接着看看`Worker`类（仅展示部分源码）：

```java
// Worker类部分源码
private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
    final Thread thread;
    Runnable firstTask;

    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this);
    }

    public void run() {
        runWorker(this);
    }
    //其余代码略...
}
```

`Worker` 类实现了 `Runnable` 接口，所以 `Worker` 也是一个线程任务。在构造方法中，创建了一个线程，线程的任务就是自己。故 `addWorker` 方法调用 `addWorker` 方法源码下半部分中的第4步 `t.start`，会触发 `Worker` 类的 `run` 方法被 JVM 调用。

再看看 `runWorker` 的逻辑：

```java
// Worker.runWorker方法源代码
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    // 1.线程启动之后，通过unlock方法释放锁
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        // 2.Worker执行firstTask或从workQueue中获取任务，如果getTask方法不返回null，循环不退出
        while (task != null || (task = getTask()) != null) {
            // 2.1进行加锁操作，保证thread不被其他线程中断（除非线程池被中断）
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            // 2.2检查线程池状态，倘若线程池处于中断状态，当前线程将中断。 
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                // 2.3执行beforeExecute 
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    // 2.4执行任务
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    // 2.5执行afterExecute方法 
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                // 2.6解锁操作
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

首先去执行创建这个 worker 时就有的任务，当执行完这个任务后，worker 的生命周期并没有结束，在 `while` 循环中，worker 会不断地调用 `getTask` 方法从**阻塞队列**中获取任务然后调用 `task.run()` 执行任务，从而达到**复用线程**的目的。只要 `getTask` 方法不返回 `null`，此线程就不会退出。

当然，核心线程池中创建的线程想要拿到阻塞队列中的任务，先要判断线程池的状态，如果 **STOP** 或者 **TERMINATED**，返回 `null`。

最后看看 `getTask()` 方法的实现：

```java
// Worker.getTask方法源码
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        // 1.allowCoreThreadTimeOut变量默认是false,核心线程即使空闲也不会被销毁
        // 如果为true,核心线程在keepAliveTime内仍空闲则会被销毁。 
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
        // 2.如果运行线程数超过了最大线程数，但是缓存队列已经空了，这时递减worker数量。 
　　　　 // 如果有设置允许线程超时或者线程数量超过了核心线程数量，
        // 并且线程在规定时间内均未poll到任务且队列为空则递减worker数量
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            // 3.如果timed为true(想想哪些情况下timed为true),则会调用workQueue的poll方法获取任务.
            // 超时时间是keepAliveTime。如果超过keepAliveTime时长，
            // poll返回了null，上边提到的while循序就会退出，线程也就执行完了。
            // 如果timed为false（allowCoreThreadTimeOut为falsefalse
            // 且wc > corePoolSize为false），则会调用workQueue的take方法阻塞在当前。
            // 队列中有任务加入时，线程被唤醒，take方法返回任务，并执行。
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

核心线程的会一直卡在 `workQueue.take` 方法，被阻塞并挂起，不会占用 CPU 资源，直到拿到 `Runnable` 然后返回（当然如果 **allowCoreThreadTimeOut** 设置为 `true`，那么核心线程就会去调用 `poll` 方法，因为 `poll` 可能会返回 `null`，所以这时候核心线程满足超时条件也会被销毁）。

非核心线程会 `workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) `，如果超时还没有拿到，下一次循环判断 **compareAndDecrementWorkerCount** 就会返回 `null`，Worker 对象的 `run()` 方法循环体的判断为 `null`，任务结束，然后线程被系统回收 。



### 3. 四种常见的线程池

`Executors` 类中提供的几个静态方法来创建线程池。

#### 3.1 newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

`CacheThreadPool` 的**运行流程**如下：

1. 提交任务进线程池。
2. 因为 **corePoolSize** 为 0 的关系，不创建核心线程，线程池最大为 Integer.MAX_VALUE。
3. 尝试将任务添加到 **SynchronousQueue** 队列。
4. 如果 SynchronousQueue 入列成功，等待被当前运行的线程空闲后拉取执行。如果当前没有空闲线程，那么就创建一个非核心线程，然后从 SynchronousQueue 拉取任务并在当前线程执行。
5. 如果 SynchronousQueue 已有任务在等待，入列操作将会阻塞。

当需要执行很多**短时间**的任务时，CacheThreadPool 的线程复用率比较高， 会显著的**提高性能**。而且线程 60s 后会回收，意味着即使没有任务进来，CacheThreadPool 并不会占用很多资源。



#### 3.2 newFixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
```

核心线程数量和总线程数量相等，都是传入的参数 nThreads，所以只能创建核心线程，不能创建非核心线程。因为 LinkedBlockingQueue 的默认大小是 Integer.MAX_VALUE，故如果核心线程空闲，则交给核心线程处理；如果核心线程不空闲，则入列等待，直到核心线程空闲。

**与 CachedThreadPool 的区别**：

- 因为 `corePoolSize == maximumPoolSize`，所以 FixedThreadPool 只会创建核心线程。 而 CachedThreadPool 因为 `corePoolSize=0`，所以只会创建非核心线程。
- 在 `getTask()` 方法，如果队列里没有任务可取，线程会一直阻塞在 `LinkedBlockingQueue.take()`，线程不会被回收。 CachedThreadPool 会在 60s 后收回。
- 由于线程不会被回收，会一直卡在阻塞，所以**没有任务的情况下， FixedThreadPool 占用资源更多**。
- 都几乎不会触发拒绝策略，但是原理不同。FixedThreadPool 是因为阻塞队列可以很大（最大为 Integer 最大值），故几乎不会触发拒绝策略；CachedThreadPool 是因为线程池很大（最大为 Integer 最大值），几乎不会导致线程数量大于最大线程数，故几乎不会触发拒绝策略。



#### 3.3 newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

有且仅有一个核心线程（ `corePoolSize == maximumPoolSize = 1`），使用了 LinkedBlockingQueue（容量很大），所以，**不会创建非核心线程**。所有任务按照**先来先执行**的顺序执行。如果这个唯一的线程不空闲，那么新来的任务会存储在任务队列里等待执行。



#### 3.4 newScheduledThreadPool

创建一个定长线程池，支持定时及周期性任务执行。

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

//ScheduledThreadPoolExecutor():
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE,
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue());
}
```

四种常见的线程池基本够我们使用了，但是《阿里巴巴开发手册》不建议我们直接使用 Executors 类中的线程池，而是建议通过 `ThreadPoolExecutor` 的方式，这样的处理方式让写的同学需要更加明确线程池的运行规则，规避资源耗尽的风险。





## **二、阻塞队列**

### 1. 什么是阻塞队列？

阻塞队列（BlockingQueue）是一个**支持两个附加操作的队列**。这两个附加的操作支持**阻塞（线程安全）的**插入和移除方法。 

- 支持阻塞的插入方法：当队列满时，队列会阻塞插入元素的线程，直到队列不满。 

- 支持阻塞的移除方法：在队列为空时，获取元素的线程会等待队列变为非空。

> BlockingQueue 是 `java util.concurrent` 包下重要的数据结构，区别于普通的队列，BlockingQueue 提供了**线程安全的队列访问方式**，并发包下很多高级同步类的实现都是基于 BlockingQueue 实现的。

BlockingQueue 一般用于**生产者-消费者**模式，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程；该模式能够简化开发过程，一方面消除了生产者类与消费者类之间的代码依赖性，另一方面将生产数据的过程与使用数据的过程解耦简化负载。。**阻塞队列 BlockingQueue 就是存放元素的容器**。



### 2. BlockingQueue 的基本操作方法

阻塞队列提供了四组不同的方法用于插入、移除、检查元素：

| 方法/处理方式 |  抛出异常  | 返回特殊值 |  一直阻塞   |      超时退出       |
| :-----------: | :--------: | :--------: | :---------: | :-----------------: |
|   插入方法    |  add (e)   | offer (e)  | **put (e)** | offer (e,time,unit) |
|   移除方法    | remove ()  |  poll ()   | **take ()** |  poll (time,unit)   |
|   检查方法    | element () |  peek ()   |      —      |          —          |

- **抛出异常**：如果试图的操作无法立即执行，抛异常。当阻塞队列满时候，再往队列里插入元素，会抛出 `IllegalStateException("Queue full")` 异常。当队列为空时，从队列里获取元素时会抛出 `NoSuchElementException` 异常。
- **返回特殊值**：如果试图的操作无法立即执行，返回一个特殊值，通常是 true / false。
- **一直阻塞**：如果试图的操作无法立即执行，则一直阻塞或者响应中断。
- **超时退出**：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功，通常是 true / false。

**注意之处：**

- 不能往阻塞队列中插入 null，会抛出空指针异常。
- 可以访问阻塞队列中的任意元素，调用 `remove(o)` 可以将队列之中的特定对象移除，但并不高效，尽量避免使用。
- 如果是无界阻塞队列，队列不可能会出现满的情况，所以使用 `put` 或 `offer` 方法永远不会被阻塞，而且使用 `offer` 方法时，该方法永远返回 true。 



### 3. BlockingQueue 的实现类

#### 3.1 ArrayBlockingQueue

由**数组**结构组成的**有界**阻塞队列。内部结构是数组，故具有数组的特性。

```java
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull = lock.newCondition();
}
```

可以初始化队列大小， 且一旦初始化不能改变。构造方法中的 `fair` 表示控制对象的内部锁是否采用公平锁，默认是**非公平锁**。

> 所谓公平访问队列是**对阻塞的线程而言的**，可以按照阻塞的先后顺序访问队列，即先阻塞线程先访问队列。非公平性是对先等待的线程是非公平的，当队列可用时，阻塞的线程都可以争夺访问队列的资格，有可能先阻塞的线程最后才访问队列。为了保证公平性，通常会降低吞吐量。



#### 3.2 LinkedBlockingQueue

由**链表**结构组成的**有界**阻塞队列。内部结构是链表，具有链表的特性。默认队列的大小是 `Integer.MAX_VALUE`，也可以指定大小。此队列按照**先进先出**的原则对元素进行排序。



#### 3.3 DelayQueue

DelayQueue 是一个**支持延时获取元素**的**无界（不限制大小，故插入不会被阻塞）阻塞队列**。队列使用 PriorityQueue 来实现。队列中的元素必须实现 `java.util.concurrent.Delayed` 接口，在创建元素时可以指定多久才能从队列中获取当前元素。**该队列中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素**。 

DelayQueue 非常有用，可以将 DelayQueue 运用在以下应用场景：

- **缓存系统的设计**：可以用 DelayQueue 保存缓存元素的有效期，使用一个线程循环查询 DelayQueue，一旦能从 DelayQueue 中获取元素时，表示缓存有效期到了。 

- **定时任务调度**：使用 DelayQueue 保存当天将会执行的任务和执行时间，一旦从 DelayQueue 中获取到任务就开始执行，比如 TimerQueue 就是使用 DelayQueue 实现的。



#### 3.4 PriorityBlockingQueue

基于**优先级**的**无界阻塞队列**（优先级的判断通过构造函数传入的 Compator 对象来决定，**默认升序自然排序**），内部控制线程同步的锁采用的是**非公平锁**。

```java
public PriorityBlockingQueue(int initialCapacity,
                                  Comparator<? super E> comparator) {
         this.lock = new ReentrantLock(); //默认构造方法-非公平锁
         ...//其余代码略
}
```

**注意：**

**PriorityBlockingQueue** 不会阻塞数据生产者（因为队列是无界的），而只会在没有可消费的数据时，阻塞数据的消费者。因此使用的时候要特别注意，**生产者生产数据的速度绝对不能快于消费者消费数据的速度，否则时间一长，会最终耗尽所有的可用堆内存空间**。对于使用默认大小的 **LinkedBlockingQueue** 也是一样的。



#### 3.5 SynchronousQueue

SynchronousQueue 是一个**不存储元素（无任何内部容量）**的**阻塞队列**。**每一个 put 操作必须等待一个 take 操作，否则不能继续添加元素，反之亦然**。 

它支持公平访问队列。**默认情况下线程采用非公平性策略访问队列**。使用以下构造方法可以创建公平性访问的 SynchronousQueue，如果设置为 true，则等待的线程会采用先进先出的顺序访问队列。

```java
public SynchronousQueue(boolean fair) {
	transferer = fair ? new TransferQueue() :new TransferStack();
}
```

SynchronousQueue 可以看成是一个传球手，负责把生产者线程处理的数据**直接传递**给消费者线程。队列本身并不存储任何元素，非常**适合传递性场景**。SynchronousQueue 的吞吐量高于 LinkedBlockingQueue 和 ArrayBlockingQueue。 

> 需要区别容量为 1 的 ArrayBlockingQueue、LinkedBlockingQueue。

以下方法的返回值，可以帮助理解这个队列：

- `iterator()`：永远返回空，因为里面没有东西。
- `peek()`：永远返回 null。
- `put()`：往 queue 放进去一个 element 以后就一直 wait，直到有其他 thread 进来把这个 element 取走。
- `offer()`：往 queue 里放一个 element 后立即返回，如果碰巧这个 element 被另一个 thread 取走了，offer 方法返回 true，认为 offer 成功；否则返回 false。
- `take()`：取出并且 remove 掉 queue 里的 element，取不到东西他会一直等。
- `poll()`：取出并且 remove 掉 queue 里的 element，只有到碰巧另外一个线程正在往 queue 里 offer 数据或者 put 数据的时候，该方法才会取到东西。否则立即返回 null。
- `isEmpty()`：永远返回 true。
- `remove() & removeAll()`：永远返回 false。



### 4. 阻塞队列的原理

> 如果队列是空的，消费者会一直等待，当生产者添加元素时，消费者是如何知道当前队列有元素的呢？JDK 是如何设计阻塞队列的，如何让生产者和消费者进行高效率的通信呢？

阻塞队列的原理很简单，利用了 Lock 锁的多条件（**Condition**）阻塞控制。接下来以 ArrayBlockingQueue JDK 1.8 的源码来分析。

首先是构造器，除了初始化队列的大小和是否是公平锁之外，还对同一个锁（lock）初始化了**两个监视器**，分别是 `notEmpty` 和 `notFull`。这两个监视器的作用目前可以简单理解为标记分组，当该线程是 `put` 操作时，给他加上监视器 `notFull`，标记这个线程是一个生产者；当线程是 `take` 操作时，给他加上监视器 `notEmpty`，标记这个线程是消费者。

```java
//数据元素数组
final Object[] items;
//下一个待取出元素索引
int takeIndex;
//下一个待添加元素索引
int putIndex;
//元素个数
int count;
//内部锁
final ReentrantLock lock;
//消费者监视器
private final Condition notEmpty;
//生产者监视器
private final Condition notFull;  

public ArrayBlockingQueue(int capacity, boolean fair) {
    //..省略其他代码
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}
```

**put 操作的源码：**

```java
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    // 1.自旋拿锁
    lock.lockInterruptibly();
    try {
        // 2.判断队列是否满了
        while (count == items.length)
            // 2.1如果满了，阻塞该线程，并标记为notFull线程，
            // 等待notFull的唤醒，唤醒之后继续执行while循环。
            notFull.await();
        // 3.如果没有满，则进入队列
        enqueue(e);
    } finally {
        lock.unlock();
    }
}
private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
    // 4 唤醒一个等待的线程
    notEmpty.signal();
}
```

总结 put 的流程：

1. 所有执行 put 操作的线程竞争 lock 锁，拿到了 lock 锁的线程进入下一步，没有拿到 lock 锁的线程自旋竞争锁。
2. 判断阻塞队列是否满了，如果满了，则调用 `await()` 方法阻塞这个线程，并标记为 `notFull`（生产者）线程，同时释放 lock 锁，等待被消费者线程唤醒。
3. 如果没有满，则调用 `enqueue(e)` 方法将元素 put 进阻塞队列。注意这一步的线程还有一种情况是第二步中阻塞的线程被唤醒且又拿到了 lock 锁的线程。
4. 唤醒一个标记为 `notEmpty`（消费者）的线程。

**take 操作的源码：**

```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            notEmpty.await();
        return dequeue();
    } finally {
        lock.unlock();
    }
}
private E dequeue() {
    // assert lock.getHoldCount() == 1;
    // assert items[takeIndex] != null;
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length)
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
    notFull.signal();
    return x;
}
```

take 操作和 put 操作的流程是类似的，总结一下 take 操作的流程：

1. 所有执行 take 操作的线程竞争 lock 锁，拿到了 lock 锁的线程进入下一步，没有拿到 lock 锁的线程自旋竞争锁。
2. 判断阻塞队列是否为空，如果是空，则调用 `await()` 方法阻塞这个线程，并标记为 `notEmpty`（消费者）线程，同时释放 lock 锁，等待被生产者线程唤醒。
3. 如果没有空，则调用 `dequeue()` 方法。注意这一步的线程还有一种情况是第二步中阻塞的线程被唤醒且又拿到了 lock 锁的线程。
4. 唤醒一个标记为 `notFull`（生产者）的线程。

**注意**

1. put 和 take 操作都需要**先获取锁**，没有获取到锁的线程会被挡在第一道大门之外自旋拿锁，直到获取到锁。
2. 就算拿到锁了之后，也**不一定**会顺利进行 put/take 操作，需要判断**队列是否可用**（是否满/空），**如果不可用，则会被阻塞，并释放锁**。
3. 在第 2 点被阻塞的线程会被唤醒，但是在唤醒之后，**依然需要拿到锁**才能继续往下执行，否则，自旋拿锁，拿到锁了再 while 判断队列是否可用。



### 5. 示例和使用场景

#### 5.1 生产者-消费者模型

```java
public class Test {
    private int queueSize = 10;
    private ArrayBlockingQueue<Integer> queue = new ArrayBlockingQueue<Integer>(queueSize);

    public static void main(String[] args) {
        Test test = new Test();
        Producer producer = test.new Producer();
        Consumer consumer = test.new Consumer();

        producer.start();
        consumer.start();
    }

    class Consumer extends Thread {
        @Override
        public void run() {
            consume();
        }

        private void consume() {
            while(true){
                try {
                    queue.take();
                    System.out.println("从队列取走一个元素，队列剩余"+queue.size()+"个元素");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    class Producer extends Thread {
        @Override
        public void run() {
            produce();
        }

        private void produce() {
            while(true){
                try {
                    queue.put(1);
                    System.out.println("向队列取中插入一个元素，队列剩余空间："+(queueSize-queue.size()));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

下面是这个例子的输出片段：

```
从队列取走一个元素，队列剩余0个元素
从队列取走一个元素，队列剩余0个元素
向队列取中插入一个元素，队列剩余空间：9
向队列取中插入一个元素，队列剩余空间：9
向队列取中插入一个元素，队列剩余空间：9
向队列取中插入一个元素，队列剩余空间：8
向队列取中插入一个元素，队列剩余空间：7
向队列取中插入一个元素，队列剩余空间：6
向队列取中插入一个元素，队列剩余空间：5
向队列取中插入一个元素，队列剩余空间：4
向队列取中插入一个元素，队列剩余空间：3
向队列取中插入一个元素，队列剩余空间：2
向队列取中插入一个元素，队列剩余空间：1
向队列取中插入一个元素，队列剩余空间：0
从队列取走一个元素，队列剩余1个元素
从队列取走一个元素，队列剩余9个元素
```

注意，这个例子中的输出结果看起来可能有问题，比如有几行在插入一个元素之后，队列的剩余空间不变。这是由于 **`System.out.println` 语句没有锁**。考虑到这样的情况：线程 1 在执行完 put/take 操作后立即失去 CPU 时间片，然后切换到线程 2 执行 put/take 操作，执行完毕后回到线程 1 的 `System.out.println` 语句并输出，发现这个时候阻塞队列的 size 已经被线程 2 改变了，所以这个时候输出的 size 并不是当时线程 1 执行完 put/take 操作之后阻塞队列的 size，但可以确保的是 size 不会超过10个。实际上使用阻塞队列是没有问题的。



#### 5.2 线程池中使用阻塞队列

```java
 public ThreadPoolExecutor(int corePoolSize,
                           int maximumPoolSize,
                           long keepAliveTime,
                           TimeUnit unit,
                           BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
}
```

Java 中的线程池就是使用阻塞队列实现的，在了解阻塞队列之后，无论是使用 Executors 类中已经提供的线程池，还是自己通过 ThreadPoolExecutor 实现线程池，都会更加得心应手。

