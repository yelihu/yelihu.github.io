---
title: AQS总成为你大厂面试的"滑铁卢"？别让它成为送命题了...
date: 2025-06-22 19:57:57
updated: 2025-06-22 19:57:57
toc: true
categories:
    - Java
    - 分布式中间件
tags:
    - AQS
    - Java
    - JUC
    - 并发
    - 多线程
    - ReentrantLock
    - synchronized
description: 本文深入浅出地剖析了Java并发编程中AQS（AbstractQueuedSynchronizer）的核心原理与设计思想，涵盖了其状态变量、线程安全机制、等待队列以及独占与共享模式等关键知识点，是理解和掌握JUC并发工具的必备指南。
author: 虎太郎
excerpt: 在这篇文章中，我将通过层层递进的思考，最终揭示AQS（AbstractQueuedSynchronizer）的精妙设计。如果你对ReentrantLock、synchronized、Semaphore等并发工具的底层实现感到困惑，那么这篇文章也许能解答一些问题。
---


# AQS总成为你大厂面试的"滑铁卢"？别让它成为送命题了...

聊聊AQS吧，你是不是觉得它有点绕，一下子说不清？什么CAS、状态变量、等待队列，听着就头大…… 有时候是不是感觉，自己明明对并发工具有点了解，但真要从头设计一个，又不知道从何说起？



## 实现一个并发工具类，需要考虑哪些问题

那如果让你从头实现一个并发工具类，你应该考虑哪些问题呢？首先得知道"并发工具类"是干什么的。

**并发工具类就是一个能让多个线程安全地去访问共享资源的工具。 注意，是"多个线程"访问"共享资源"。**



那么，每个线程来访问时，就必须判断当前的共享资源是否正在被占用。

- 如果没有线程在访问，那当前线程就可以去访问。
- 如果已经有线程正在访问，那当前线程要么重试、要么阻塞、要么直接放弃。

这立刻就引出了第一个要解决的问题。



### 先找个变量state - 标记资源的使用

在思考如何表示共享资源"正在被占用"或"空闲"时，首先想到了引入一个状态变量 state。

- state 为 0 时，表示资源空闲。
- state 为 1 时，表示资源被占用。

这其实就是 ReentrantLock 的基本思想。换一个角度，是否用布尔值（boolean）来表示 state 会更简单明了呢？

这里提一个场景来拒绝boolean的设计，同一个共享资源，某个线程可能需要多次地访问。每次访问，state 就要加一；每释放一次，state 就要减一。直到 state 减到 0，才代表资源被彻底释放为空闲状态。

这就是 ReentrantLock 中"可重入"的精妙设计。考虑到上面这几种情况，其实我们就会发现在ReentrantLock的设计里面， state 不能是简单的布尔类型，必须设置为 int 类型。

如果共享资源可以同时被好几个线程访问呢？比如，某个资源最多支持3个线程同时访问。那这时候 state 的值最大就应该为 。这便是 Semaphore（信号量）的设计理念。



> 总结：state我们设计的宽容度越好，能表示的语义的能力越强。state在不同场景下承担不同的语义，从而支持了AQS框架的通用性和扩展性。



### 将state设置为volatile - 可见性很重要

多个线程都会访问和修改 state 值，你必须保证两件事：

1. 可见性：一个线程对 state 的修改，其他线程必须能立刻看到。怎么办？很简单，给 state 变量加上 volatile 关键字。
1. 原子性：多个线程同时修改 state 值时，不能出现数据错乱。怎么办？使用 CAS (Compare-And-Swap) 操作去修改 state，以保证其原子性。

现在，当一个线程想访问共享资源时，它通过 CAS 去修改 state 的状态。修改成功了，就代表获取了资源，可以继续访问。那……修改失败了怎么办呢？



### 获取资源失败的线程如何处理？

一直自旋（Spinning）重试吗？如果并发量高，长时间的自旋会极大地浪费CPU资源，AQS的做法把这些修改失败的线程阻塞住，等资源空闲了再唤醒它们。

AQS把每个等待的线程包装成一个 Node 节点，然后用这些节点组成一个双向链表，基于这个链表实现一个先进先出 (FIFO) 的等待队列，通过这种设计 **高效地管理这些因为竞争失败需要等待的线程**。

- 当线程 CAS 修改 state 状态失败后，就将该线程包装成 Node 加入队列并阻塞自己。
- 当持有资源的线程释放资源后，就去队列里唤醒头部的节点，让它重新尝试获取资源。





## 从AQS源码看API设计

我们设计的框架有了，那具体的 API 应该怎么设计呢？AQS在设计的时候，考虑到上层业务可能会用两种方式调用它：

- 调用方法1——尝试获取：某个业务，他只是想"尝试"获取一下共享资源，如果**获取不到，就立刻去做其他操作**，不想等待。针对这种场景，可以设计一个方法叫 tryAcquire()，返回 true 或 false 代表成功或失败。

- 调用方法2——阻塞获取：另一种场景是，**业务"必须"取到共享资源才能继续操作，否则就一直等待**。那就可以设计一个方法叫 acquire()，没有返回值，获取不到就阻塞，直到其他线程释放资源并且自己获取成功为止。




来看看 AQS 的源码是怎么设计的。AQS 中有一个核心方法 tryAcquire(int arg):

```java
// AQS.java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

tryAcquire() 定义了获取共享资源的方式。但奇怪的是，它内部没有实现，而是直接抛出了一个异常。问题来了，为什么这么设计？

原因是： AQS 只是一个抽象的、通用的并发工具框架。不同的场景（如独占锁、共享锁）获取资源的方式是不同的。因此，AQS 并不去定义具体的获取方式，而是把这个权力开放给上层业务（它的子类比如ReentrantLock）去重写该方法。

AQS 只定义一件事：获取资源成功了没有（即 tryAcquire 返回 true 还是 false）。

再来看另一个核心方法 acquire(int arg)：

```java
// AQS.java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```




这段代码的逻辑非常清晰：先调用（子类实现的）tryAcquire() 去获取资源。如果获取不到，就通过 addWaiter() 将当前线程加入等待队列，并通过 acquireQueued() 将其阻塞。

>  这里其实也是模板方法模式的一个经典应用，实际实现是依靠子类的方法重写来体现的。





## AQS的实用智慧：理解Java并发工具的奥秘

了解了 AQS 的设计思想后，你会发现很多关于Java并发工具的问题，一下子就清晰起来了！

### ReentrantLock 和 synchronized 有什么区别？

#### 区别1：尝试非阻塞获取锁

ReentrantLock 是基于 AQS 实现的。AQS 提供了 tryAcquire() 机制，所以 ReentrantLock 可以很自然地实现 tryLock() 方法，允许线程尝试加锁，如果失败可以立即返回去做其他操作。而 synchronized 是做不到的，它只有获取不到锁就一直阻塞。



> 调用 ReentrantLock 的 tryLock() 方法时，它会内部调用其同步器（Sync，FairSync 或 NonfairSync）实现的 tryAcquire() 方法。这个 tryAcquire() 方法会尝试以非阻塞的方式获取锁，意味着获取不到就直接返回给上层业务代码了，这是synchronized无法支持的。



#### 区别2：公平性

刚刚我们提到，AQS 是基于一个双向链表实现了一个 FIFO 等待队列。你有没有想过为什么用队列？除了插入删除效率高（O(1)）之外，一个重要原因就是为了实现公平策略。

队列天然就是"先进先出"的，这为实现公平锁提供了基础。AQS 的子类只需要重写 tryAcquire() 方法，就可以在内部自定义实现公平或非公平策略。

ReentrantLock 恰好就是这么做的。它内部有两个子类：

1. FairSync（公平锁）：严格依赖 AQS 的队列结构，实现了"先来后到"的获取锁策略。
1. NonfairSync（非公平锁）：允许新来的线程"插队"抢锁，而不管队列中是否有等待者。


> FairSync 在 tryAcquire() 中增加了对等待队列的检查，以强制公平性；而 NonfairSync 则直接尝试获取锁，允许更激进的抢占策略以优化性能。



所以这里总结另外一个区别，就是：ReentrantLock 基于 AQS 的等待队列，可以灵活地选择实现公平锁和非公平锁。而 **synchronized 始终都是非公平的**。



#### 区别3：更强大的线程通信

上一期我们讲过，synchronized 的监视器（Monitor）中有两个集合，一个是 EntryList（锁池），用来存放获取锁失败的线程；另一个是 WaitSet（等待池），用来存放调用了 wait() 方法主动放弃锁的线程。

那么，使用了 ReentrantLock 之后，没有拿到锁的线程被放到了 AQS 的等待队列中。那如果拿到锁的线程也需要等待某个条件满足（例如，等待资源到位），调用 await() 方法主动放弃锁之后，这个线程应该被放到哪里呢？

这就涉及到了另外一个对象：Condition。ReentrantLock的实例lock可以用lock.newCondition()给自己创建一个条件。

```java
private final ReentrantLock lock = new ReentrantLock();
// 条件A：用于管理等待特定事件A的线程组
private final Condition conditionA = lock.newCondition();
// 条件B：用于管理等待特定事件B的线程组
private final Condition conditionB = lock.newCondition();
```

考虑一个消息队列的场景，队列中既有“高优先级消息”，也有“普通优先级消息”。那么我们就可以new出来highPriorityCondition 和 normalPriorityCondition两个Condition。另外，同一个锁（比如一个装水果的篮子）很可能消费的线程的关注点是不同的（比如isEmpty / isFull），我们区分不同的Condition能够分离这些不同关注点的线程。

Condition 对象用来管理"条件队列"。调用了 await() 的线程，就会被放到 Condition 自己的条件队列中。你还可以 new 多个 Condition 对象，配合不同 Condition 对象的 signal() 或 signalAll() 方法，去实现更加精细（比如处理高优和低优）、有针对性的唤醒通知。

使用 synchronized，所有的等待线程都挤在一个 WaitSet 里，我们可能需要在业务代码里对锁的状态进行if else判断来确定“为什么会被唤醒”，但是 ReentrantLock + Condition的代码中，我们就有有 isFull.await()（等锁对象‘满’）、isEmpty.await()（等锁对象‘空’）、dataAvailable.await() 等可读性非常高的代码出现代码逻辑变得非常清晰，因为每个 await 和 signal 都直接关联到它所表示的业务条件。这使得代码更容易理解、调试和维护。



这又是 ReentrantLock 和 synchronized 的一个巨大不同。synchronized 只有一个等待池，notifyAll() 会唤醒所有等待线程，而 ReentrantLock 借助 Condition 可以实现**分组唤醒**。这又是个区别。



## AQS的两种模式：独占与共享

AQS 有两种工作模式，这也体现了它的灵活性：

### 独占模式 (Exclusive Mode)

只允许**单个线程独占资源**，适用于互斥的场景，典型的例子就是锁，比如 ReentrantLock。

- 通过 acquire() 和 tryAcquire() 获取资源。
- state 为 0 时表示资源空闲。加锁时通过 CAS 将 state 修改为 1。
- state 还可以代表"重入次数"。重入一次 state 加 1，释放一次 state 减 1，减到 0 才算完全释放。
- 加锁失败的线程进入 AQS 的等待队列。



### 共享模式 (Shared Mode)

允许**多个线程可以共享同一个资源**，适用于多线程协作的场景，比如 Semaphore (信号量) 或 CountDownLatch (倒计时门闩)。

对于 Semaphore：state 就代表"许可证"的数量。比如，某个资源可以同时被 3 个线程访问，那 state 就初始化为 3。每有一个线程获取资源，state 就递减 1。当 state 减到 0 时，新的线程就无法获取资源，必须进入 AQS 等待队列。当有线程释放资源时，state 再递增。代码简略如下：

```java
// java.util.concurrent.Semaphore.Sync
abstract static class Sync extends AbstractQueuedSynchronizer {
    // ...
    protected int tryAcquireShared(int permits) {
        for (;;) { // 自旋，直到成功或返回
            int available = getState(); // 获取当前可用的许可证数量
            int remaining = available - permits; // 计算获取后剩余的许可证数量
            if (remaining < 0 || compareAndSetState(available, remaining))
                return remaining; // 如果不足或CAS成功，则返回剩余数量
        }
    }
    protected boolean tryReleaseShared(int releases) {
      for (;;) { // 自旋，直到成功
          int current = getState(); // 获取当前许可证数量
          int next = current + releases; // 释放后新的许可证数量
          if (next < current) // 溢出检查，通常不会发生
              throw new Error("Maximum permit count exceeded");
          if (compareAndSetState(current, next))
              return true; // CAS成功，释放成功
      }
    }
}
```



对于 CountDownLatch：state 代表"剩余的任务数"。主线程调用 await() 方法等待，工作线程每完成一个任务就调用 countDown() 方法使 state 递减。当 state 归零时，AQS 会唤醒所有在 await() 上等待的主线程。

```java
// java.util.concurrent.CountDownLatch.Sync
private static final class Sync extends AbstractQueuedSynchronizer {
    // ...
    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1; // 如果state为0，返回1表示成功，否则返回-1表示失败
    }
  
    // ...
    protected boolean tryReleaseShared(int releases) {
        for (;;) { // 自旋，直到成功
            int c = getState(); // 获取当前剩余任务数
            if (c == 0) // 如果已经为0，则无需再减
                return false;
            int nextc = c - 1; // 任务数减一
            if (compareAndSetState(c, nextc)) // 原子更新state
                return nextc == 0; // 如果更新后state为0，返回true表示可以唤醒等待线程
        }
    }
}
```

当 state 最终减到 0 时，tryReleaseShared 返回 true，AQS 就会唤醒所有在共享模式下等待的线程。这个唤醒的操作可以在AQS的源码里面一窥究竟。

如果 tryReleaseShared 返回 true，表示资源已经成功释放并且可能需要唤醒等待者，那么 AQS 就会紧接着调用 doReleaseShared() 方法。

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer
public final boolean releaseShared(int arg) {
    // 调用子类（如Semaphore或CountDownLatch）重写的tryReleaseShared方法
    if (tryReleaseShared(arg)) { // 如果子类的释放操作成功
        doReleaseShared(); // 则执行AQS内部的共享模式释放逻辑（唤醒线程），**查看下面代码**
        return true;
    }
    return false;
}

// java.util.concurrent.locks.AbstractQueuedSynchronizer
private void doReleaseShared() {
    /*
     * 确保至少一个节点被唤醒。
     * 在共享模式下，一个被唤醒的线程如果成功获取资源，
     * 可能会继续唤醒其后的节点（"传播"行为），直到没有需要唤醒的或队列为空。
     */
    for (;;) {
        Node h = head; // 获取等待队列的头节点
        if (h != null && h != tail) { // 如果队列不为空且有实际等待的节点
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) { // 如果头节点的状态是SIGNAL（表示它的后继节点需要被唤醒）
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0)) // 尝试清除SIGNAL状态
                    continue; // CAS失败，重试
                unparkSuccessor(h); // 唤醒头节点的后继节点
            }
            // else if (ws == 0 && !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
            //   continue;                // 共享模式的传播机制，这里简化省略
        }
        if (h == head) // 头节点没有变化，退出循环
            break;
    }
}
```

doReleaseShared() 方法负责检查 AQS 等待队列的头部节点。如果头节点的状态是 SIGNAL (一个特殊的等待状态，表示该节点的后继节点需要被唤醒)，它就会调用 unparkSuccessor(h) 来唤醒头节点的后继线程。

unparkSuccessor(Node node) 方法会找到 node 的后继节点，并使用 LockSupport.unpark() 来唤醒对应的线程，这些线程将会参与对信号量等锁资源的竞争。





## 说回来，为什么Java不直接用操作系统级的锁呢？

了解了这么多之后，你有没有想过，Java 为什么不直接使用操作系统的 mutex（互斥量）来限制多线程对共享资源的访问，而是要自己在 JVM 层面费这么大劲定义 AQS 这么多东西呢？

性能开销：如果直接使用操作系统的 mutex（如 Linux 上的 futex 或 Windows 上的 Critical Section 等），每次加锁和解锁都涉及到系统调用。mutex 作为一种低级同步原语，直接作用于这些共享资源，因此其管理必须在具备最高权限的内核态进行，以维护系统稳定性和安全性。那么每次线程**尝试获取锁或释放锁（对应tryAcquire/tryRelease）**，都可能触发一次系统调用。在高并发场景下，频繁的系统调用会导致大量的用户态到内核态的切换，从而成为性能瓶颈。

AQS 通过在用户态使用 CAS 操作和高效的队列管理，将**锁的竞争，也就是tryAcquire/tryRelease都是用户态的CAS，和等待队列处理逻辑放在用户态完成**，能够极大地减少这种开销，只有在线程确实需要阻塞时，才会触发底层的内核级操作。

> 怎么样触发底层内核操作？
>
> park() 的实现最终会调用操作系统的相应系统调用，如刚刚提到的 Linux 上的 futex 。因为参与竞争失败了，所以将当前线程挂起park，将自己封装成一个 Node 节点，并加入到 AQS 的等待队列中。直到被 unpark() 或者被中断。
>
> 操作系统内核接收到系统调用后，会将该线程设置为 WAITING 或 BLOCKED 状态，并将其从 CPU 调度队列中移除，从而释放 CPU 资源给其他可运行的线程。
>
> 当持有锁的线程释放锁，或者某个条件满足需要唤醒等待线程时，它会调用 LockSupport.unpark(Thread t)。unpark() 也会发起一个系统调用，通知操作系统唤醒目标线程。操作系统将该线程从等待队列中移出，并设置为 RUNNABLE 状态。这个线程会重新获得CPU时间片，获得时间片之后第一件事就是去继续tryAcquire() **(精神可嘉，俺也要像线程一样！💪)**



灵活性与可扩展性：AQS 是一个抽象的并发操作框架。它允许开发者通过继承，用纯 Java 代码非常灵活地去自定义公平策略、实现共享或独占模式等等。但操作系统的 mutex，我们是无法在 Java 层面去做这种灵活拓展的。



所以，AQS 的存在，是 Java 在性能和灵活性之间做出的一个卓越设计。



## 附录：一些示例Demo - 唤醒对AQS实现类使用上的记忆



```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.Semaphore;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class ConcurrencyDemo {

    // --- 仓库管理：使用 ReentrantLock 和 Condition ---
    static class Warehouse {
        private final ReentrantLock lock = new ReentrantLock();
        private final Queue<String> products = new LinkedList<>();
        private final int CAPACITY = 5;

        // 条件：当仓库满了，生产者等待
        private final Condition notFull = lock.newCondition();
        // 条件：当有A零件时，A零件消费者等待
        private final Condition notEmptyA = lock.newCondition();
        // 条件：当有B零件时，B零件消费者等待
        private final Condition notEmptyB = lock.newCondition();

        public void put(String product) throws InterruptedException {
            lock.lock(); // 获取 ReentrantLock 确保互斥访问
            try {
                while (products.size() == CAPACITY) {
                    System.out.println(Thread.currentThread().getName() + "：仓库已满，等待空位...");
                    notFull.await(); // 生产者等待，释放锁，进入 notFull 条件队列
                }
                products.offer(product);
                System.out.println(Thread.currentThread().getName() + "：生产了 [" + product + "]。当前仓库：" + products);

                // 生产后，根据产品类型唤醒对应的消费者
                if (product.startsWith("A")) {
                    notEmptyA.signalAll(); // 唤醒所有等待A零件的消费者
                } else if (product.startsWith("B")) {
                    notEmptyB.signalAll(); // 唤醒所有等待B零件的消费者
                }
                // 也可以唤醒所有，但这里演示精细化唤醒
                // notEmptyA.signalAll();
                // notEmptyB.signalAll();

            } finally {
                lock.unlock(); // 释放 ReentrantLock
            }
        }

        public String takeA() throws InterruptedException {
            lock.lock();
            try {
                // 消费者A只关心A零件
                while (products.isEmpty() || !products.peek().startsWith("A")) {
                    System.out.println(Thread.currentThread().getName() + "：没有A零件，等待...");
                    notEmptyA.await(); // 消费者A等待，释放锁，进入 notEmptyA 条件队列
                }
                String product = products.poll();
                System.out.println(Thread.currentThread().getName() + "：消费了 [" + product + "]。当前仓库：" + products);
                notFull.signalAll(); // 消费后，唤醒等待空位的生产者
                return product;
            } finally {
                lock.unlock();
            }
        }

        public String takeB() throws InterruptedException {
            lock.lock();
            try {
                // 消费者B只关心B零件
                while (products.isEmpty() || !products.peek().startsWith("B")) {
                    System.out.println(Thread.currentThread().getName() + "：没有B零件，等待...");
                    notEmptyB.await(); // 消费者B等待，释放锁，进入 notEmptyB 条件队列
                }
                String product = products.poll();
                System.out.println(Thread.currentThread().getName() + "：消费了 [" + product + "]。当前仓库：" + products);
                notFull.signalAll(); // 消费后，唤醒等待空位的生产者
                return product;
            } finally {
                lock.unlock();
            }
        }
    }

    // --- 运输部门：使用 Semaphore 限制并发装载 ---
    static class TruckLoader {
        // Semaphore 限制同时只能有 3 个装载许可
        private final Semaphore loadingPermits = new Semaphore(3);

        public void loadTruck(String productName) throws InterruptedException {
            loadingPermits.acquire(); // 获取一个装载许可，如果没有则阻塞
            try {
                System.out.println(Thread.currentThread().getName() + "：开始装载 [" + productName + "] 到卡车...");
                Thread.sleep(500); // 模拟装载耗时
                System.out.println(Thread.currentThread().getName() + "：完成装载 [" + productName + "]。");
            } finally {
                loadingPermits.release(); // 释放装载许可
            }
        }
    }

    public static void main(String[] args) {
        Warehouse warehouse = new Warehouse();
        TruckLoader loader = new TruckLoader();

        // 生产者A：生产A零件
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    warehouse.put("A-Part-" + i);
                    Thread.sleep(80);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Factory-A").start();

        // 生产者B：生产B零件
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    warehouse.put("B-Part-" + i);
                    Thread.sleep(120);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Factory-B").start();

        // 消费者A：只消费A零件
        new Thread(() -> {
            try {
                for (int i = 0; i < 8; i++) { // 假设只消费8个
                    String part = warehouse.takeA();
                    loader.loadTruck(part); // 消费后进行装载
                    Thread.sleep(150);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Consumer-A").start();

        // 消费者B：只消费B零件
        new Thread(() -> {
            try {
                for (int i = 0; i < 8; i++) { // 假设只消费8个
                    String part = warehouse.takeB();
                    loader.loadTruck(part); // 消费后进行装载
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Consumer-B").start();

        // 额外的消费者A，演示多个线程等待同一Condition
        new Thread(() -> {
            try {
                for (int i = 0; i < 4; i++) { // 假设再消费4个
                    String part = warehouse.takeA();
                    loader.loadTruck(part);
                    Thread.sleep(200);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Consumer-A-Extra").start();
    }
}

```