# **并发的源头**

## **源头之一：缓存导致的可见性问题**

### 单核时代

![https://tva1.sinaimg.cn/large/00831rSTly1gdk18hw6ivj30vq0hswla.jpg](https://tva1.sinaimg.cn/large/008eGmZEly1gnhnc15g20j30vq0hsack.jpg)

### 多核时代

![https://tva1.sinaimg.cn/large/00831rSTly1gdk19xa23uj30vq0hs10y.jpg](https://tva1.sinaimg.cn/large/008eGmZEly1gnhnbxi4pzj30vq0hsmz8.jpg)

## **源头之二：线程切换带来的原子性问题**

操作系统允许某个进程执行一小段时间，例如 50 毫秒，过了 50 毫秒操作系统就会重新选择一个进程来执行（我们称为“任务切换”），这个 50 毫秒称为“**时间片**”。

![https://tva1.sinaimg.cn/large/00831rSTly1gdk1fgw2twj30vq0hsgn0.jpg](https://tva1.sinaimg.cn/large/008eGmZEly1gnhnc1jumaj30vq0hsjrx.jpg)

## **源头之三：编译优化带来的有序性问题**

![https://tva1.sinaimg.cn/large/00831rSTly1gdk1ssotakj30vq0hstct.jpg](https://tva1.sinaimg.cn/large/008eGmZEly1gnhnc23dwsj30vq0hsdhp.jpg)

# 线程的生命周期

## **通用生命周期**

通用的线程生命周期基本上可以用下图这个“五态模型”来描述。这五态分别是：**初始状态、可运行状态、运行状态、休眠状态**和**终止状态**。

![https://tva1.sinaimg.cn/large/00831rSTgy1gdm751yl2ej30vq0jumy8.jpg](https://tva1.sinaimg.cn/large/008eGmZEly1gnhnc062ilj30vq0ju3z0.jpg)

这“五态模型”的详细情况如下所示。

1. **初始状态**，指的是线程已经被创建，但是还不允许分配 CPU 执行。这个状态属于编程语言特有的，不过这里所谓的被创建，仅仅是在编程语言层面被创建，而在操作系统层面，真正的线程还没有创建。
2. **可运行状态**，指的是线程可以分配 CPU 执行。在这种状态下，真正的操作系统线程已经被成功创建了，所以可以分配 CPU 执行。
3. 当有空闲的 CPU 时，操作系统会将其分配给一个处于可运行状态的线程，被分配到 CPU 的线程的状态就转换成了**运行状态**。
4. 运行状态的线程如果调用一个阻塞的 API（例如以阻塞方式读文件）或者等待某个事件（例如条件变量），那么线程的状态就会转换到**休眠状态**，同时释放 CPU 使用权，休眠状态的线程永远没有机会获得 CPU 使用权。当等待的事件出现了，线程就会从休眠状态转换到可运行状态。
5. 线程执行完或者出现异常就会进入**终止状态**，终止状态的线程不会切换到其他任何状态，进入终止状态也就意味着线程的生命周期结束了。

## **Java 中线程的生命周期**

Java 语言中线程共有六种状态，分别是：

1. NEW（初始化状态）
2. RUNNABLE（可运行 / 运行状态）
3. BLOCKED（阻塞状态）
4. WAITING（无时限等待）
5. TIMED_WAITING（有时限等待）
6. TERMINATED（终止状态）

这看上去挺复杂的，状态类型也比较多。但其实在操作系统层面，Java 线程中的 BLOCKED、WAITING、TIMED_WAITING 是一种状态，即前面我们提到的休眠状态。也就是说**只要 Java 线程处于这三种状态之一，那么这个线程就永远没有 CPU 的使用权**。

所以 Java 线程的生命周期可以简化为下图：

![https://tva1.sinaimg.cn/large/00831rSTgy1gdm7c0x45rj30vq0ju0ty.jpg](https://tva1.sinaimg.cn/large/008eGmZEly1gnhnbz7ijij30vq0ju0tc.jpg)

## **方法是如何被执行的**

```
int a = 7；
int[] b = fibonacci(a);
int[] c = b;
```

![https://tva1.sinaimg.cn/large/00831rSTly1gdmc3str9nj30vq0kljtd.jpg](https://tva1.sinaimg.cn/large/008eGmZEly1gnhnc0noclj30vq0klgms.jpg)

# 线程通信

## 等待通知机制 wait()、notify()、notifyAll()

有一些需要注意:

- wait() 、notify()、notifyAll() 调用的前提都是获得了对象的锁(也可称为对象监视器)。
- 调用 wait() 方法后线程会释放锁，进入 `WAITING` 状态，该线程也会被移动到**等待队列**中。
- 调用 notify() 方法会将**等待队列**中的线程移动到**同步队列**中，线程状态也会更新为 `BLOCKED`
- 从 wait() 方法返回的前提是调用 notify() 方法的线程释放锁，wait() 方法的线程获得锁。

等待通知有着一个经典范式：

线程 A 作为消费者：

1. 获取对象的锁。
2. 进入 while(判断条件)，并调用 wait() 方法。
3. 当条件满足跳出循环执行具体处理逻辑。

线程 B 作为生产者:

1. 获取对象锁。
2. 更改与线程 A 共用的判断条件。
3. 调用 notify() 方法。

伪代码如下:

```java
//Thread A

synchronized(Object){
    while(条件){
        Object.wait();
    }
    //do something
}

//Thread B
synchronized(Object){
    条件=false;//改变条件
    Object.notify();
}
```

## Semaphore 信号量

使用场景：同一时间控制线程并发数量

```java
    public static void main(String[] args) {
        int N = 8;            //工人数
        Semaphore semaphore = new Semaphore(1); //机器数目
        for (int i = 0; i < N; i++) {
            int finalI = i;
            new Thread(() -> {
                try {
                    semaphore.acquire();  // 在子线程里控制资源占用
                    System.out.println("工人" + finalI + "占用一个机器在生产...");
                    Thread.sleep(1000);
                    System.out.println("工人" + finalI + "释放出机器");
                    semaphore.release();   // 在子线程里控制释放资源占用
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
```

## join()方法

在 t1.join() 时会一直阻塞到 t1 执行完毕，所以最终主线程会等待 t1 线程执行完毕。

其实从源码可以看出，join() 也是利用的等待通知机制：

核心逻辑:

```java
    while (isAlive()) {
        wait(0);
    }
```

在 join 线程完成后会调用 notifyAll() 方法，是在 JVM 实现中调用，所以这里看不出来。

## volatile共享内存

采用 volatile 修饰主要是为了内存可见性

## CountDownLatch

```java
public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(10);
        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(() -> {
                synchronized (CountDownLatchDemo.class) {
                    System.out.println("id:" + finalI + "," + Thread.currentThread().getName());
                    //latch.countDown();
                    System.out.println("线程任务" + finalI + "结束，其他任务继续");
                    countDownLatch.countDown();
                }
            }).start();
        }
        countDownLatch.await(); // 注意跟CyclicBarrier不同，这里在主线程await
        System.out.println("==>主线程执行结束。。。。");
    }
```

CountDownLatch 也是基于 AQS(AbstractQueuedSynchronizer) 实现的，初始化数量，然后递减直到为0。

- 初始化一个 CountDownLatch 时告诉并发的线程，然后在每个线程处理完毕之后调用 countDown() 方法。
- 该方法会将 AQS 内置的一个 state 状态 -1 。
- 最终在主线程调用 await() 方法，它会阻塞直到 `state == 0` 的时候返回。

## CyclicBarrier

```java
public static void main(String[] args) throws InterruptedException {

        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, () -> {
            System.out.println("回调>>" + Thread.currentThread().getName());
            System.out.println("回调>>线程执行结束");
        });

        for (int i = 0; i < 5; i++) {
            int finalI = i;
            new Thread(() -> {
                System.out.println("id:" + finalI + "," + Thread.currentThread().getName());
                try {
                    System.out.println("线程组任务" + finalI + "结束，其他任务继续");
                    cyclicBarrier.await();   // 注意跟CountDownLatch不同，这里在子线程await
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }).start();
        }

        System.out.println("==>主线程执行结束。。。。");
    }
```

CyclicBarrier 中文名叫做屏障或者是栅栏，也可以用于线程间通信。CyclicBarrier可以重复利用。它可以等待 N 个线程都达到某个状态后继续运行的效果。

- 首先初始化线程参与者。
- 调用 `await()` 将会在所有参与者线程都调用之前等待。
- 直到所有参与者都调用了 `await()` 后，所有线程从 `await()` 返回继续后续逻辑。

## 线程响应中断

可以采用中断线程的方式来通信，调用了 `thread.interrupt()` 方法其实就是将 thread 中的一个标志属性置为了 true。

并不是说调用了该方法就可以中断线程，如果不对这个标志进行响应其实是没有什么作用(这里对这个标志进行了判断)。

**但是如果抛出了 InterruptedException 异常，该标志就会被 JVM 重置为 false。**

## 线程池awaitTermination()方法

如果是用线程池来管理线程，可以使用以下方式来让主线程等待线程池中所有任务执行完毕:

```java
    private static void executorService() throws Exception{
        BlockingQueue<Runnable> queue = new LinkedBlockingQueue<>(10) ;
        ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(5,5,1, TimeUnit.MILLISECONDS,queue) ;
        poolExecutor.execute(new Runnable() {
            @Override
            public void run() {
                LOGGER.info("running");
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        poolExecutor.execute(new Runnable() {
            @Override
            public void run() {
                LOGGER.info("running2");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        poolExecutor.shutdown();
        while (!poolExecutor.awaitTermination(1,TimeUnit.SECONDS)){
            LOGGER.info("线程还在执行。。。");
        }
        LOGGER.info("main over");
    }
```

输出结果:

```
2018-03-16 20:18:01.273 [pool-1-thread-2] INFO  c.c.actual.ThreadCommunication - running2
2018-03-16 20:18:01.273 [pool-1-thread-1] INFO  c.c.actual.ThreadCommunication - running
2018-03-16 20:18:02.273 [main] INFO  c.c.actual.ThreadCommunication - 线程还在执行。。。
2018-03-16 20:18:03.278 [main] INFO  c.c.actual.ThreadCommunication - 线程还在执行。。。
2018-03-16 20:18:04.278 [main] INFO  c.c.actual.ThreadCommunication - main over
```

使用这个 `awaitTermination()` 方法的前提需要关闭线程池，如调用了 `shutdown()` 方法。

调用了 `shutdown()` 之后线程池会停止接受新任务，并且会平滑的关闭线程池中现有的任务。

## 管道通信

```java
    public static void piped() throws IOException {
        //面向于字符 PipedInputStream 面向于字节
        PipedWriter writer = new PipedWriter();
        PipedReader reader = new PipedReader();

        //输入输出流建立连接
        writer.connect(reader);
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                LOGGER.info("running");
                try {
                    for (int i = 0; i < 10; i++) {

                        writer.write(i+"");
                        Thread.sleep(10);
                    }
                } catch (Exception e) {

                } finally {
                    try {
                        writer.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                LOGGER.info("running2");
                int msg = 0;
                try {
                    while ((msg = reader.read()) != -1) {
                        LOGGER.info("msg={}", (char) msg);
                    }
                } catch (Exception e) {
                }
            }
        });
        t1.start();
        t2.start();
    }
```

输出结果:

```
2018-03-16 19:56:43.014 [Thread-0] INFO  c.c.actual.ThreadCommunication - running
2018-03-16 19:56:43.014 [Thread-1] INFO  c.c.actual.ThreadCommunication - running2
2018-03-16 19:56:43.130 [Thread-1] INFO  c.c.actual.ThreadCommunication - msg=0
2018-03-16 19:56:43.132 [Thread-1] INFO  c.c.actual.ThreadCommunication - msg=1
2018-03-16 19:56:43.132 [Thread-1] INFO  c.c.actual.ThreadCommunication - msg=2
2018-03-16 19:56:43.133 [Thread-1] INFO  c.c.actual.ThreadCommunication - msg=3
2018-03-16 19:56:43.133 [Thread-1] INFO  c.c.actual.ThreadCommunication - msg=4
2018-03-16 19:56:43.133 [Thread-1] INFO  c.c.actual.ThreadCommunication - msg=5
2018-03-16 19:56:43.133 [Thread-1] INFO  c.c.actual.ThreadCommunication - msg=6
2018-03-16 19:56:43.134 [Thread-1] INFO  c.c.actual.ThreadCommunication - msg=7
2018-03-16 19:56:43.134 [Thread-1] INFO  c.c.actual.ThreadCommunication - msg=8
2018-03-16 19:56:43.134 [Thread-1] INFO  c.c.actual.ThreadCommunication - msg=9
```

Java 虽说是基于内存通信的，但也可以使用管道通信。

需要注意的是，输入流和输出流需要首先建立连接。这样线程 B 就可以收到线程 A 发出的消息了。

实际开发中可以灵活根据需求选择最适合的线程通信方式。

# 锁原理

## 锁分类

### 同一进程内

- synchronized （隐式锁，加锁、释放锁都不需要手动操作）
- 重入锁 ReentrantLock （显示锁）
- 读写锁 ReentrantReadWriteLock （显示锁）

### 不同进程内（分布式锁）

- 基于数据库
- reids分布式锁
- zk分布式锁

## **简单锁模型**

![https://tva1.sinaimg.cn/large/00831rSTly1gdl71l90uij30vq0hs3zh.jpg](https://tva1.sinaimg.cn/large/008eGmZEly1gnhnbwflicj30vq0hsmxk.jpg)

## **改进后锁模型**

![https://tva1.sinaimg.cn/large/00831rSTly1gdl72drxk2j30vq0hswg5.jpg](https://tva1.sinaimg.cn/large/008eGmZEly1gnhnc2k0gqj30vq0hsgmi.jpg)

## **Java 语言提供的锁技术：synchronized**

![https://tva1.sinaimg.cn/large/00831rSTly1gdl73ribnvj30vq0hs0uq.jpg](https://tva1.sinaimg.cn/large/008eGmZEly1gnhnby9y3aj30vq0hs3zn.jpg)

### **synchronized的使用方式有三种**

- 静态方法（锁类）
- 普通方法（锁类对象）
- 代码块（锁类对象）

### **synchronized的实现原理**

**jvm 层面：**

synchronized依赖于monitorenter和monitorexit两个指令实现的。

- monitorenter

  每个对象都有一把锁，当一个线程进入同步代码块，都会去获取这个对象所持有monitor对象锁（C++实现），如果当前线程获取锁，会把monitor对象进入数自增1次。如果该线程重复进入，会把monitor对象进入数再次自增1次。

  当有其他线程进入，会把其他线程放入等待队列排队，直到获取锁的线程将monitor对象的进入数设置为0释放锁，其他线程才有机会获取锁。

- monitorexit

**synchronized优化层面：**

线程竞争锁会引起用户态和内核态的频繁切换，造成资源浪费且效率不高，在jdk1.6以后synchronized做了性能优化。

https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e4c75b2c2fc46edab356ceca24bae00~tplv-k3u1fbpfcp-zoom-1.image

- 无锁

  如果不加synchronized关键字，表示无锁，很好理解。

- 偏向锁

  升级过程：当线程进入同步块时，Markword会存储偏向线程的id并且cas将Markword锁状态标识为01，是否偏向用1表示当前处于偏向锁（对着上图来看），如果是偏向线程下次进入同步代码只要比较Markword的线程id是否和当前线程id相等，如果相等不用做任何操作就可以进入同步代码执行，如果比较后不相等说明有其他线程竞争锁，synchronized会升级成轻量级锁。这个过程中在操作系统层面不用做内核态和用户态的切换，减少切换线程带来的资源消耗。

  膨胀过程：当有另外线程进入，偏向锁会升级成轻量级锁。比如线程A是偏向锁，这是B线程进入，就会成轻量级锁， **只要有两个线程就会升级成轻量级锁** 。

- 轻量级锁

  升级过程：在线程运行获取锁后，会在栈帧中创造锁记录并将MarkWord复制到锁记录，然后将MarkWord指向锁记录，如果当前线程持有锁，其他线程再进入，此时其他线程会cas自旋，直到获取锁，轻量级锁适合多线程交替执行，效率高（cas只消耗cpu）。

  膨胀过程：有两种情况会膨胀成重量级锁。1种情况是cas自旋10次还没获取锁。第2种情况其他线程正在cas获取锁，第三个线程竞争获取锁，锁也会膨胀变成重量级锁。

- 重量级锁

  重量级锁升级后是不可逆的，也就是说重量锁不可以再变为轻量级锁。

  ### **[适应性自旋](https://crossoverjie.top/JCSprout/#/thread/Synchronize?id=适应性自旋)**

  在使用 `CAS` 时，如果操作失败，`CAS` 会自旋再次尝试。由于自旋是需要消耗 `CPU` 资源的，所以如果长期自旋就白白浪费了 `CPU`。`JDK1.6`加入了适应性自旋,如果某个锁自旋很少成功获得，那么下一次就会减少自旋。

# ReentrantLock 实现原理

ReentrantLock是可重入锁，主要基于AQS来实现公平锁和非公平锁，获取锁的时候通过CAS修改AQS的state字段，默认为0，线程获取到锁后会更新成1，并且设置独占线程为当前线程，如果更新成功代表当前线程拿到了锁。由于支持可重入，所以同一个线程拿锁state会继续增加，在释放锁的时候要将state减到0才算释放成功，并唤醒其他线程。

## 锁类型

ReentrantLock 分为公平锁和非公平锁，可以通过构造方法来指定具体类型

```java
/**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
				// 默认是非公平锁
        sync = new NonfairSync();
    }

    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

使用方式：

```java
private ReentrantLock lock = new ReentrantLock();
    public void run() {
        lock.lock();
        try {
            //do bussiness
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
```

## 公平锁实现原理

```java
public void lock() {
        sync.acquire(1);
    }

public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

/**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;
        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        @ReservedStackAccess
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

- 通过ReentrantLock构造方法设置为公平锁
- 通过tryAcquire方法尝试获取锁
- 首先会判断 AQS 中的 state 是否等于 0，0表示目前没有其他线程获得锁，当前线程就可以尝试获取锁。尝试之前会利用 hasQueuedPredecessors() 方法来判断 AQS 的队列中中是否有其他线程，如果有则不会尝试获取锁(这是公平锁特有的情况)。
- 如果队列中没有线程就利用 CAS 来将 AQS 中的 state 修改为1，也就是获取锁，获取成功则将当前线程置为获得锁的独占线程(setExclusiveOwnerThread(current))。
- 如果 state 大于 0 时，说明锁已经被获取了，则需要判断获取锁的线程是否为当前线程(ReentrantLock 支持重入)，是则需要将 state + 1，并将值更新。
- 如果 tryAcquire(arg) 获取锁失败，则需要用 addWaiter(Node.EXCLUSIVE) 将当前线程写入队列中。

## 非公平锁实现原理

```java
public boolean tryLock() {
        return sync.nonfairTryAcquire(1);
    }

/**
         * Performs non-fair tryLock.  tryAcquire is implemented in
         * subclasses, but both need nonfair try for trylock method.
         */
        @ReservedStackAccess
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

- 通过nonfairTryAcquire方法获取锁
- 首先会判断 AQS 中的 state 是否等于 0，0表示目前没有其他线程获得锁，当前线程就可以尝试获取锁
- 如果队列中没有线程就利用 CAS 来将 AQS 中的 state 修改为1，也就是获取锁，获取成功则将当前线程置为获得锁的独占线程(setExclusiveOwnerThread(current))。
- 如果 state 大于 0 时，说明锁已经被获取了，则需要判断获取锁的线程是否为当前线程(ReentrantLock 支持重入)，是则需要将 state + 1，并将值更新。

## 释放锁原理

```java
public void unlock() {
        sync.release(1);
    }

public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

@ReservedStackAccess
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```

- 公平锁和非公平锁的释放流程都是一样的
- 首先会判断当前线程是否为获得锁的线程，由于是重入锁所以需要将 state 减到 0 才认为完全释放锁。
- 释放之后需要调用 unparkSuccessor(h) 来唤醒被挂起的线程

# **线程池总结**

## 线程池的拒绝策略

- 丢弃任务抛出RejectedExecutionException异常，这个也是线程池的默认拒绝策略。
- 丢弃任务，但是不抛出异常。
- 丢弃队列最前面的任务，然后重新提交被拒绝的任务。
- 由调用线程(提交任务的线程)处理该任务。

## 创建线程池

- Executors.newFixedThreadPool

  ```java
  public static ExecutorService newFixedThreadPool(int nThreads) {
          return new ThreadPoolExecutor(nThreads, nThreads,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>());
      }
  ```

  创建一个线程池，该线程池重用在共享的无边界队列上运行的固定数量的线程。 在任何时候，最多nThreads线程都是活动的处理任务。 如果在所有线程都处于活动状态时提交了其他任务，则它们将在队列中等待，直到某个线程可用为止。 如果在关闭之前执行过程中由于执行失败导致任何线程终止，则在执行后续任务时将使用新线程代替。 池中的线程将一直存在，直到将其显式shutdown为止。

- Executors.newSingleThreadExecutor()

  ```java
  public static ExecutorService newSingleThreadExecutor() {
          return new FinalizableDelegatedExecutorService
              (new ThreadPoolExecutor(1, 1,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>()));
      }
  ```

  创建一个执行程序，该执行程序使用在不受限制的队列上操作的单个工作线程。 （但是请注意，如果该单线程由于在关闭之前的执行期间失败而终止，则在需要执行新任务时将替换为新线程。）保证任务按顺序执行，并且活动的任务不超过一个在任何给定时间。 与其他等效的newFixedThreadPool(1)不同，保证返回的执行程序不能重新配置为使用其他线程。

- Executors.newScheduledThreadPool

  ```java
  public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
          return new ScheduledThreadPoolExecutor(corePoolSize);
      }
  
  public ScheduledThreadPoolExecutor(int corePoolSize) {
          super(corePoolSize, Integer.MAX_VALUE,
                DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
                new DelayedWorkQueue());
      }
  ```

  创建一个线程池，该线程池可以安排命令在给定的延迟后运行或定期执行

- Executors.newCachedThreadPool()

  ```java
  public static ExecutorService newCachedThreadPool() {
          return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                        60L, TimeUnit.SECONDS,
                                        new SynchronousQueue<Runnable>());
      }
  ```

  创建一个线程池，该线程池根据需要创建新线程，但是将在先前构造的线程可用时重用它们。 这些池通常将提高执行许多短暂的异步任务的程序的性能。 如果可用， execute将重用以前构造的线程。 如果没有可用的现有线程，则将创建一个新线程并将其添加到池中。 六十秒内未使用的线程将终止并从缓存中删除。 因此，保持空闲时间足够长的池不会消耗任何资源。 请注意，可以使用ThreadPoolExecutor构造函数创建具有相似属性但不同细节（例如，超时参数）的ThreadPoolExecutor 。