# 2. Java 并发机制的底层实现原理

Java 代码在编译后会变成 Java 字节码，字节码被类加载器加载到 JVM 里，JVM 执行字节码，最终需要转化为汇编指令在 CPU 上执行。

**Java 中所使用的并发机制依赖于 JVM 的实现和 CPU 的指令。**

实现 Java 并发有两个重要的关键字：

- **volatile**
- **synchronized**



## 2.1 volatile 的应用

`volatile` 是**轻量级**的 `synchronized`，它在多处理器开发中保证了共享变量的“**可见性**”（当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。）。

> 如果 volatile 变量修饰符使用恰当的话，它比 synchronized 的使用和执行成本更低，因为它**不会引起线程上下文的切换和调度**。



### 2.1.1 volatile 的定义与实现原理

Java 编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过**排他锁**单独获得这个变量。

如果一个字段被声明成 `volatile`，Java 线程内存模型确保**所有线程看到这个变量的值是一致的**。 

下面是与 `volatile` 实现原理相关的 CPU 术语与说明：

![1636698669463](./imgs/1636698669463.png)



- volatile 是如何来保证可见性的呢？

为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2 或其他）后再进行操作，<u>但操作完不知道何时会写到内存</u>。

如果对声明了 volatile 的变量进行写操作，**JVM** 就会向处理器发送一条 **Lock 前缀的指令**，**将这个变量所在缓存行的数据写回到系统内存**。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。

所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现**缓存一致性协议**——**每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。**



volatile 的两条实现原则：

- **Lock** **前缀指令会引起处理器缓存回写到内存。**

  Lock 前缀指令导致在执行指令期间，声言处理器的 **LOCK#信号**。在多处理器环境中，**LOCK#信号确保在声言该信号期间，处理器可以独占任何共享内存**【因为 LOCK#信号会锁住总线，导致其他 CPU 不能访问总线，不能访问总线就意味着不能访问系统内存】。

  但是，在最近的处理器里，LOCK＃信号一般不锁总线，而是锁缓存，毕竟锁总线开销的比较大。

  在目前的处理器中，如果访问的内存区域已经缓存在处理器内部，则不会声言 LOCK#信号。相反，它会**锁定这块内存区域的缓存并回写到内存，并使用缓存一致性机制来确保修改的原子性**，此操作被称为“**缓存锁定**”，**缓存一致性机制会阻止同时修改由两个以上处理器缓存的内存区域数据。**

- **一个处理器的缓存回写到内存会导致其他处理器的缓存无效。**

  IA-32 处理器和 Intel 64 处理器使用 `MESI（修改、独占、共享、无效）控制协议` 去维护内部缓存和其他处理器缓存的一致性。在多核处理器系统中进行操作的时候，IA-32 和 Intel 64 处理器能嗅探其他处理器访问系统内存和它们的内部缓存。**处理器使用嗅探技术保证它的内部缓存、系统内存和其他处理器的缓存的数据在总线上保持一致。**



### 2.1.2 volatile 的使用优化

这里举的是一个 JDK 7 并发包中的一个队列集合类 `LinkedTransferQueue` 的例子，它在使用 volatile 变量时，用一种**追加字节**的方式来优化队列出队和入队的性能。

LinkedTransferQueue 代码：

![1636700464354](./imgs/1636700464354.png)

> LinkedTransferQueue 使用一个内部类类型（PaddedAtomicReference）来定义队列的头节点（head）和尾节点（tail），而这个内部类相对于父类 AtomicReference 只做了一件事情，就是将共享变量追加到 64 字节。

为什么追加 64 字节能够提高并发编程的效率呢？

因为许多处理器的 L1、L2 或 L3 缓存的高速缓存行是 64 个字节宽，不支持部分填充缓存行，这意味着，如果队列的头节点和尾节点都不足 64 字节的话，处理器会将它们都读到同一个高速缓存行中，在多处理器下每个处理器都会缓存同样的头、尾节点，当一个处理器试图修改头节点时，会将整个缓存行锁定，那么在缓存一致性机制的作用下，会导致其他处理器不能访问自己高速缓存中的尾节点，而队列的入队和出队操作则需要不停修改头节点和尾节点，所以在多处理器的情况下将会严重影响到队列的入队和出队效率。使用追加到 64 字节的方式来**填满高速缓冲区的缓存行，避免头节点和尾节点加载到同一个缓存行，使头、尾节点在修改时不会互相锁定。**

***

在以下两种场景下不应该使用追加字节的方式：

- **缓存行非 64 字节宽的处理器**

  如 P6 系列和奔腾处理器，它们的 L1 和 L2 高速缓存 行是 32 个字节宽。 

- **共享变量不会被频繁地写**

  因为使用追加字节的方式需要处理器读取更多的字节到高速缓冲区，这本身就会带来一定的性能消耗，如果共享变量不被频繁写的话，锁的几率也非常小，就没必要通过追加字节的方式来避免相互锁定。



## 2.2 synchronized 的实现原理与应用

`synchronized` 实现同步的基础：**Java 中的每一个对象都可以作为锁。** 

具体表现为：

- 对于**普通同步方法**，锁是**当前实例对象**。 

- 对于**静态同步方法**，锁是**当前类的 Class 对象**。 

- 对于**同步方法块**，锁是 **Synchonized 括号里配置的对象**。

当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。

 

Synchronized 在 JVM 里的实现原理：

> JVM 基于进入和退出 **Monitor 对象**来实现方法同步和代码块同步，但两者的实现细节不一样。代码块同步是使用 **`monitorenter`** 和 **`monitorexit`** 指令实现的，而方法同步是使用另外一种方式实现的，细节在 JVM 规范里并没有详细说明。但是，方法的同步同样可以使用这两个指令来实现。
>
> - **monitorenter 指令**在编译后插入到同步代码块的开始位置。
> - **monitorexit 指令**插入到方法结束处和异常处。
>
> JVM 要保证**每个 monitorenter 必须有对应的 monitorexit 与之配对。任何对象都有一个 monitor 与之关联，当且一个 monitor 被持有后，它将处于锁定状态。线程执行到 monitorenter 指令时，将会尝试获取对象所对应的 monitor 的所有权，即尝试获得对象的锁。**



锁是存在哪里的？锁里面会存储什么信息？

### 2.2.1 Java 对象头

**synchronized 用的锁是存在 Java 对象头里的。**

如果对象是**数组类型**，则虚拟机用 **3 个字宽**（Word）存储对象头，如果对象是**非数组类型**，则用 **2 字宽**存储对象头。在 32 位虚拟机中，**1 字宽 = 4 字节，即 32bit。**

Java 对象头的内容和对应长度如下表：

![1636716072513](./imgs/1636716072513.png)

Java 对象头里的 Mark Word 里默认存储对象的 **HashCode**、**分代年龄**和**锁标记位**。32 位 JVM 的 Mark Word 的默认存储结构如下表所示：

![1636716270682](./imgs/1636716270682.png)

在运行期间，Mark Word 里存储的数据会随着锁标志位的变化而变化，其 4 种数据状态如下：

![1636716440648](./imgs/1636716440648.png)

64 位虚拟机下，Mark Word 是 64 bit 大小的，其存储结构如下：

![1636716543228](./imgs/1636716543228.png)



### 2.2.2 锁的升级与对比

在 Java SE 1.6 中，锁一共有 4 种状态，级别**从低到高**依次是：**无锁**状态、**偏向锁**状态、**轻量级锁**状态和**重量级锁**状态，这几个状态会随着竞争情况逐渐升级。

**锁可以升级但不能降级**，意味着偏向锁升级成轻量级锁后不能降级成偏向锁。这种锁升级却不能降级的策略，目的是为了**提高获得锁和释放锁的效率**。

#### 1. 偏向锁

大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了**偏向锁**。

当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程 ID，以后该线程在进入和退出同步块时不需要进行 CAS 操作来加锁和解锁，只需简单地测试一下对象头的 Mark Word 里是否存储着指向当前线程的偏向锁。

如果测试成功，表示线程已经获得了锁。如果测试失败，则需要再测试一下 Mark Word 中偏向锁的标识是否设置成 1（表示当前是偏向锁）：如果没有设置，则使用 CAS 竞争锁；如果设置了，则尝试使用 CAS 将对象头的偏向锁指向当前线程。

##### 偏向锁的撤销

偏向锁使用了一种**等到竞争出现才释放锁**的机制，所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁。

偏向锁的撤销，需要等待**全局安全点**（在这个时间点上没有正在执行的字节码）。

> 它会首先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否活着，如果线程不处于活动状态，则将对象头设置成无锁状态；如果线程仍然活着，拥有偏向锁的栈会被执行，遍历偏向对象的锁记录，栈中的锁记录和对象头的 Mark Word 要么重新偏向于其他线程，要么恢复到无锁或者标记对象不适合作为偏向锁，最后唤醒暂停的线程。

下图中的线程 1 演示了偏向锁初始化的流程，线程 2 演示了偏向锁撤销的流程：

![1636717170442](./imgs/1636717170442.png)



##### 关闭偏向锁

偏向锁在 Java 6 和 Java 7 里是默认启用的，但是它在应用程序启动几秒钟之后才激活，如有必要可以使用 JVM 参数来关闭延迟：`-XX:BiasedLockingStartupDelay=0` 。如果确定应用程序里所有的锁通常情况下处于竞争状态，可以通过 JVM 参数关闭偏向锁：`-XX:- UseBiasedLocking=false`，那么程序默认会进入轻量级锁状态。 



#### 2. 轻量级锁

##### 轻量级锁加锁

线程在执行同步块之前，JVM 会**先在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的 Mark Word 复制到锁记录中**，官方称为 `Displaced Mark Word`。

然后线程尝试使用 CAS 将对象头中的 Mark Word 替换为**指向锁记录的指针**。如果成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用**自旋**来获取锁。

##### 轻量级锁解锁

轻量级解锁时，会使用原子的 CAS 操作将 Displaced Mark Word 替换回到对象头，如果成功，则表示没有竞争发生。如果失败，表示当前锁存在竞争，锁就会膨胀成重量级锁。

下图是两个线程同时争夺锁，导致锁膨胀的流程图：

![1636718630572](./imgs/1636718630572.png)

因为自旋会消耗 CPU，为了避免无用的自旋（比如获得锁的线程被阻塞住了），**一旦锁升级成重量级锁，就不会再恢复到轻量级锁状态。当锁处于这个状态下，其他线程试图获取锁时，都会被阻塞住**，当持有锁的线程释放锁之后会唤醒这些线程，被唤醒的线程就会进行新一轮的锁争夺。



#### 3. 锁的优缺点对比

![1636718788045](./imgs/1636718788045.png)



## 2.3 原子操作的实现原理

原子操作（atomic operation）意为“**不可被中断的一个或一系列操作**”。

下面是与原子操作相关的几个术语：

![1636720909700](./imgs/1636720909700.png)

> 比较并交换即 CAS	



### 2.3.1 处理器如何实现原子操作

处理器提供**总线锁定**和**缓存锁定**两个机制来保证复杂内存操作的原子性。

#### 1. 使用总线锁保证原子性

> 如果多个处理器同时对共享变量进行读改写操作（i++就是经典的读改写操作），那么共享变量就会被多个处理器同时进行操作，这样读改写操作就不是原子的，操作完之后共享变量的值会和期望的不一致。

所谓总线锁就是**使用处理器提供的一个 LOCK＃信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住，那么该处理器可以独占共享内存。**



#### 2. 使用缓存锁保证原子性

**在同一时刻，我们只需保证对某个内存地址的操作是原子性即可**，但总线锁定把 CPU 和内存之间的通信锁住了，这使得锁定期间，其他处理器不能操作其他内存地址的数据，所以总线锁定的开销比较大，目前处理器在某些场合下使用缓存锁定代替总线锁定来进行优化。

频繁使用的内存会缓存在处理器的 L1、L2 和 L3 高速缓存里，那么原子操作就可以直接在处理器内部缓存中进行，并不需要声明总线锁。

所谓“缓存锁定”是指**内存区域如果被缓存在处理器的缓存行中，并且在 Lock 操作期间被锁定，那么当它执行锁操作回写到内存时，处理器不在总线上声言 LOCK＃信号，而是修改内部的内存地址，并允许它的缓存一致性机制来保证操作的原子性，因为缓存一致性机制会阻止同时修改由两个以上处理器缓存的内存区域数据，当其他处理器回写已被锁定的缓存行的数据时，会使缓存行无效。**



**处理器不会使用缓存锁的情况：**

1. 当操作的数据不能被缓存在处理器内部，或操作的数据跨多个缓存行（cache line）时，则处理器会调用总线锁定。 
2. 有些处理器不支持缓存锁定。



### 2.3.2 Java 如何实现原子操作

在 Java 中可以通过**锁**和**循环 CAS** 的方式来实现原子操作。

#### 1. 使用循环 CAS 实现原子操作

JVM 中的 CAS 操作正是利用了处理器提供的 `CMPXCHG` 指令实现的。

**自旋 CAS** 实现的基本思路就是**循环进行 CAS 操作直到成功为止。**

以下代码实现了一个基于 CAS 线程安全的计数器方法 safeCount 和一个非线程安全的计数器 count：

![1636721987493](./imgs/1636721987493.png)

> 以上代码运行后会发现 CAS 实现的线程安全计数器能够准确将 i 累加到 10000，而非线程安全的计数器每次运行结果都不同。



##### CAS 实现原子操作的三大问题

- **ABA 问题**

  因为 CAS 需要在操作值的时候，检查值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是 A，变成了 B，又变成了 A，那么使用 CAS 进行检查时会发现它的值没有发生变化，但是实际上却变化了。

  ABA 问题的解决思路就是**使用版本号**。在变量前面追加上版本号，每次变量更新的时候把版本号加 1，那么A→B→A 就会变成 1A→2B→3A。

- **循环时间长开销大**

  自旋 CAS 如果长时间不成功，会给 CPU 带来非常大的执行开销。

  > 如果 JVM 能支持处理器提供的 `pause` 指令，那么效率会有一定的提升。`pause` 指令有两个作用：① 它可以延迟流水线执行指令（de-pipeline），使 CPU 不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零；② 它可以避免在退出循环的时候因内存顺序冲突（Memory Order Violation）而引起 CPU 流水线被清空（CPU Pipeline Flush），从而提高 CPU 的执行效率。 

- **只能保证一个共享变量的原子操作**

  当对一个共享变量执行操作时，我们可以使用循环 CAS 的方式来保证原子操作，但是对多个共享变量操作时，循环 CAS 就无法保证操作的原子性，这个时候就可以用锁。



#### 2. 使用锁机制实现原子操作

锁机制保证了**只有获得锁的线程才能够操作锁定的内存区域。**

JVM 实现锁的方式都用了**循环 CAS**，即当一个线程想进入同步块的时候使用循环 CAS 的方式来获取锁，当它退出同步块的时候使用循环 CAS 释放锁