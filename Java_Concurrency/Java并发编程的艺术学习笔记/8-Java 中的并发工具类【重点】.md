# 8. Java 中的并发工具类【重点】

在 JDK 的并发包里提供了几个并发工具类：**CountDownLatch**、**CyclicBarrier** 和 **Semaphore** 工具类提供了一种并发流程控制的手段，**Exchanger** 工具类则提供了在线程间交换数据的一种手段。



## 8.1 等待多线程完成的 CountDownLatch

**CountDownLatch 允许一个或多个线程等待其他线程完成操作。** 

假如有这样一个需求：我们需要解析一个 Excel 里多个 sheet 的数据，此时可以考虑使用多线程，每个线程解析一个 sheet 里的数据，等到所有的 sheet 都解析完之后，程序需要提示解析完成。在这个需求中，要实现主线程等待所有线程完成 sheet 的解析操作，最简单的做法是使用 `join()` 方法，如代码清单所示：

>  JoinCountDownLatchTest.java：

```java
public class JoinCountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        Thread parser1 = new Thread(new Runnable() {
            @Override
            public void run() {
            }
        });
        Thread parser2 = new Thread(new Runnable() {
            @Override
            public void run() {
            	System.out.println("parser2 finish");
            }
        });
        parser1.start();
        parser2.start();
        parser1.join();
        parser2.join();
        System.out.println("all parser finish");
    } 
}
```

`join` 用于**让当前执行线程等待 join 线程执行结束**。其实现原理是不停检查 join 线程是否存活，如果 join 线程存活则让当前线程永远等待。其中，`wait(0)` 表示永远等待下去，代码片段如下：

```java
while (isAlive()) {
	wait(0);
}
```

直到 join 线程中止后，线程的 `this.notifyAll()` 方法会被调用，调用 `notifyAll()` 方法是在 JVM 里实现的，所以在 JDK 里看不到，可以查看 JVM 源码。 



在 JDK 1.5 之后的并发包中提供的 CountDownLatch 也可以实现 join 的功能，并且比 join 的功能更多，如代码清单所示：

> CountDownLatchTest.java：

```java
public class CountDownLatchTest {
    staticCountDownLatch c = new CountDownLatch(2);

    public static void main(String[] args) throws InterruptedException {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(1);
                c.countDown();
                System.out.println(2);
                c.countDown();
            }
        }).start();
        c.await();
        System.out.println("3");
    } 
}
```

CountDownLatch 的构造函数接收一个 int 类型的参数作为计数器，如果你想等待 N 个点完成，这里就传入 N。当我们调用 CountDownLatch 的 `countDown` 方法时，N 就会减 1，CountDownLatch 的 `await` 方法会阻塞当前线程，直到 N 变成零。由于 `countDown` 方法可以用在任何地方，所以这里说的 N 个点，可以是 N 个线程，也可以是 1 个线程里的 N 个执行步骤。用在多个线程时，只需要把这个 CountDownLatch 的引用传递到线程里即可。 



如果有某个解析 sheet 的线程处理得比较慢，我们不可能让主线程一直等待，所以可以使用另外一个带指定时间的 `await` 方法——`await(long time, TimeUnit unit)`，这个方法等待特定时间后，就会不再阻塞当前线程。`join` 也有类似的方法。 

>**注意** 
>
>计数器必须大于等于 0，只是等于 0 时候，计数器就是零，调用 `await` 方法时不会阻塞当前线程。CountDownLatch 不可能重新初始化或者修改 CountDownLatch 对象的内部计数器的值。一个线程调用 `countDown` 方法 happen-before，另外一个线程调用 `await` 方法。



## 8.2 同步屏障 CyclicBarrier

**CyclicBarrier 让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。** 



### 8.2.1 CyclicBarrier 简介

CyclicBarrier 默认的构造方法是 `CyclicBarrier(int parties)`，其参数表示屏障拦截的线程数量，每个线程调用 `await` 方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞。示例代码如代码清单所示：

> CyclicBarrierTest.java：

```java
public class CyclicBarrierTest {
    staticCyclicBarrier c = new CyclicBarrier(2);
    
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                	c.await();
                } catch (Exception e) {
                }
                System.out.println(1);
            }
        }).start();
        try {
        	c.await();
        } catch (Exception e) {
        }
        System.out.println(2);
    } 
}
```

因为主线程和子线程的调度是由 CPU 决定的，两个线程都有可能先执行，所以会产生两种输出，第一种可能输出如下：

```
1
2
```

第二种如下：

```
2
1
```

> 如果把 `new CyclicBarrier(2)` 修改成 `new CyclicBarrier(3)`，则主线程和子线程会永远等待，因为没有第三个线程执行 `await` 方法，即没有第三个线程到达屏障，所以之前到达屏障的两个线程都不会继续执行。



CyclicBarrier 还提供一个更高级的构造函数 `CyclicBarrier(int parties, Runnable barrier-Action)`，用于在线程到达屏障时，优先执行 barrierAction，方便处理更复杂的业务场景，如代码清单所示：

> CyclicBarrierTest2.java：

```java
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierTest2 {
    static CyclicBarrier c = new CyclicBarrier(2, new A());
    
    public static void main(String[] args) {
        new Thread(new Runnable() {
        @Override
        public void run() {
            try {
            	c.await();
            } catch (Exception e) {
            }
            System.out.println(1);
        }
        }).start();
        try {
        	c.await();
        } catch (Exception e) {
        }
        System.out.println(2);
    }
    
    static class A implements Runnable {
        @Override
        public void run() {
        	System.out.println(3);
        }
    } 
}
```

因为 CyclicBarrier 设置了拦截线程的数量是 2，所以必须等代码中的第一个线程和线程 A 都执行完之后，才会继续执行主线程，然后输出 2，所以代码执行后的输出如下：

```
3
1
2
```



### 8.2.2 CyclicBarrier 的应用场景

**CyclicBarrier 可以用于多线程计算数据，最后合并计算结果的场景。**

例如，用一个 Excel 保存了用户所有银行流水，每个 Sheet 保存一个账户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个 sheet 里的银行流水，都执行完之后，得到每个 sheet 的日均银行流水，最后，再用 barrierAction 用这些线程的计算结果，计算出整个 Excel 的日均银行流水，如代码清单所示：

> BankWaterService.java：

```java
/**
* 银行流水处理服务类
*/
public class BankWaterService implements Runnable {
    /**
    * 创建 4 个屏障，处理完之后执行当前类的 run 方法
    */
    private CyclicBarrier c = new CyclicBarrier(4, this);
    /**
    * 假设只有 4 个 sheet，所以只启动 4 个线程
    */
    private Executor executor = Executors.newFixedThreadPool(4);
    /**
    * 保存每个 sheet 计算出的银流结果
    */
    private ConcurrentHashMap<String, Integer> sheetBankWaterCount = new
    ConcurrentHashMap<String, Integer>();
    
    private void count() {
        for (int i = 0; i < 4; i++) {
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    // 计算当前 sheet 的银流数据，计算代码省略
                    sheetBankWaterCount
                     .put(Thread.currentThread().getName(), 1);
                    // 银流计算完成，插入一个屏障
                    try {
                    	c.await();
                    } catch (InterruptedException |
                    BrokenBarrierException e) {
                    	e.printStackTrace();
                    }
                }
            });
    	}
    }
    
    @Override
    public void run() {
        int result = 0;
        // 汇总每个 sheet 计算出的结果
        for (Entry<String, Integer> sheet : sheetBankWaterCount.entrySet()) {
        	result += sheet.getValue();
        }
        // 将结果输出 sheetBankWaterCount.put("result", result);
        System.out.println(result);
    }
    
    public static void main(String[] args) {
        BankWaterService bankWaterCount = new BankWaterService();
        bankWaterCount.count();
    } 
}
```

使用线程池创建 4 个线程，分别计算每个 sheet 里的数据，每个 sheet 计算结果是 1，再由 BankWaterService 线程汇总 4 个 sheet 计算出的结果，输出结果如下：

```
1
```



### 8.2.3 CyclicBarrier 和 CountDownLatch 的区别

**CountDownLatch 的计数器只能使用一次，而 CyclicBarrier 的计数器可以使用 `reset()` 方法重置。**所以 **CyclicBarrier 能处理更为复杂的业务场景**。例如，如果计算发生错误，可以重置计数器，并让线程重新执行一次。 

CyclicBarrier 还提供其他有用的方法，比如 `getNumberWaiting` 方法可以获得 Cyclic-Barrier 阻塞的线程数量。`isBroken()` 方法用来了解阻塞的线程是否被中断。其中 isBroken 的使用代码如代码清单所示：

> CyclicBarrierTest3.java：

```java
public class CyclicBarrierTest3 {
    staticCyclicBarrier c = new CyclicBarrier(2);
    
    public static void main(String[] args) throws InterruptedException, 
    BrokenBarrierException {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    c.await();
                } catch (Exception e) {
                }
            }
        });
        thread.start();
        thread.interrupt();
        try {
        	c.await();
        } catch (Exception e) {
        	System.out.println(c.isBroken());
        }
    } 
}
```

输出如下所示： 

```
True
```



## 8.3 控制并发线程数的 Semaphore

**Semaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源。** 

> 如何理解 Semaphore？
>
> 把它比作是控制流量的红绿灯。比如××马路要限制流量，只允许同时有一百辆车在这条路上行驶，其他的都必须在路口等待，所以前一百辆车会看到绿灯，可以开进这条马路，后面的车会看到红灯，不能驶入××马路，但是如果前一百辆中有 5 辆车已经离开了××马路，那么后面就允许有 5 辆车驶入马路，这个例子里说的车就是线程，驶入马路就表示线程在执行，离开马路就表示线程执行完成，看见红灯就表示线程被阻塞，不能执行。 



### 8.3.1 应用场景

**Semaphore 可以用于做流量控制，特别是公用资源有限的应用场景，比如数据库连接。**

假如有一个需求，要读取几万个文件的数据，因为都是 IO 密集型任务，我们可以启动几十个线程并发地读取，但是如果读到内存后，还需要存储到数据库中，而数据库的连接数只有 10 个，这时我们必须控制只有 10 个线程同时获取数据库连接保存数据，否则会报错无法获取数据库连接。这个时候，就可以使用 Semaphore 来做流量控制，如代码清单所示：

> SemaphoreTest.java：

```java
public class SemaphoreTest {
    private static final int THREAD_COUNT = 30;
    private static ExecutorService threadPool = 
    Executors.newFixedThreadPool(THREAD_COUNT);
    private static Semaphore s = new Semaphore(10);
    
    public static void main(String[] args) {
        for (int i = 0; i < THREAD_COUNT; i++) {
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    s.acquire();
                    System.out.println("save data");
                    s.release();
                } catch (InterruptedException e) {
                }
            }
            });
        }
        threadPool.shutdown();
    } 
}
```

在代码中，虽然有 30 个线程在执行，但是只允许 10 个并发执行。Semaphore 的构造方法 `Semaphore(int permits)`接受一个整型的数字，表示可用的许可证数量。`Semaphore(10)`表示允许 10 个线程获取许可证，也就是最大并发数是 10。Semaphore 的用法也很简单，首先线程使用 Semaphore 的 `acquire()`方法获取一个许可证，使用完之后调用 `release()` 方法归还许可证。还可以用 `tryAcquire()` 方法尝试获取许可证。



### 8.3.2 其他方法

Semaphore 还提供一些其他方法，具体如下：

- `intavailablePermits()`：返回此信号量中当前可用的许可证数。 

- `intgetQueueLength()`：返回正在等待获取许可证的线程数。 

- `booleanhasQueuedThreads()`：是否有线程正在等待获取许可证。 

- `void reducePermits(int reduction)`：减少 reduction 个许可证，是个 protected 方法。
- `Collection getQueuedThreads()`：返回所有等待获取许可证的线程集合，是个 protected 方法。 



## 8.4 线程间交换数据的 Exchanger

**Exchanger（交换者）是一个用于线程间协作的工具类。Exchanger 用于进行线程间的数据交换。**它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过 `exchange` 方法交换数据，如果第一个线程先执行 `exchange()` 方法，它会一直等待第二个线程也执行 `exchange` 方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

下面来看一下 Exchanger 的应用场景。

Exchanger 可以用于**遗传算法**，遗传算法里需要选出两个人作为交配对象，这时候会交换两人的数据，并使用交叉规则得出 2 个交配结果。Exchanger 也可以用于**校对工作**，比如我们需要将纸制银行流水通过人工的方式录入成电子银行流水，为了避免错误，采用 AB 岗两人进行录入，录入到 Excel 之后，系统需要加载这两个 Excel，并对两个 Excel 数据进行校对，看看是否录入一致，代码如代码清单所示：

> ExchangerTest.java：

```java
public class ExchangerTest {
    private static final Exchanger<String> exgr = new Exchanger<String>();
    private static ExecutorService threadPool = Executors.newFixedThreadPool(2);
    
    public static void main(String[] args) {
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    String A = "银行流水 A"; // A 录入银行流水数据
                    exgr.exchange(A);
                } catch (InterruptedException e) {
                }
            }
    	});
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    String B = "银行流水 B"; // B 录入银行流水数据
                    String A = exgr.exchange("B");
                    System.out.println("A 和 B 数据是否一致：" + A.equals(B) + "，A 录入的是："
                     + A + "，B 录入是：" + B);
                } catch (InterruptedException e) {
                }
            }
        });
        threadPool.shutdown();
    } 
}
```

如果两个线程有一个没有执行 `exchange()` 方法，则会一直等待，如果担心有特殊情况发生，避免一直等待，可以使用 `exchange(V x, longtimeout, TimeUnit unit)` 设置最大等待时长。 