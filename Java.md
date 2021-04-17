## 一、IO

### 1.1 概述

- **同步与异步**

消息通信机制，是一种通信状态。立即返回（无结果）or主动等待结果再返回。

- **阻塞与非阻塞**

关注的是我是否会因为等待结果而让出cpu。

### 1.2 IO的字符流和字节流

### 1.3 NIO（同步、非阻塞）

**三个重要组成部分：** Channel、Buffer、Selector

- **Channel**

双工通信（区别于流的单向性，通道是双向的）、可以异步读写、必须通过buffer读写

- **Buffer**

四步骤：write to buffer -> flip -> read from buffer -> clear

- **Selector**

一个selector管理多个channel，以减少线程间的切换

### 1.4 AIO（异步、非阻塞）

基于事件和回调机制实现的。（貌似应用不广泛）

## 二、集合（list、set、map）

### 2.1 map

#### ConcurrentHashMap的put过程

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

### 三、反射与代理

### 3.1 反射

**Core：** 在java中每个类也是对象，有点类似于linux一切皆文件，Java一切皆对象。反射就是把java类中的各种成分映射成一个个的Java对象。

#### 3.1.1 获取对象的三种方式

- 通过object类的getClass()函数，每一类都有这个函数
- 每一个类（包括基本数据类型，注意这里基本数据类型不用转成包装类）都有一个class属性，静态属性，通过类名直接访问
- 通过Class类的静态方法forName(String className)

#### 3.1.2 获取方法

- 获取构造函数
  - getConstructor 根据参数获得公有构造函数
  - getDeclaredConstructor 根据参数获得任意构造函数
  - newInstance执行构造函数，私有构造函数需要开放权限
- 获得成员变量
  - getDeclaredFields()获得所有字段
  - getFields()所有公有字段
- 获取类方法
  - getMethods()获取所有公有方法(包括继承的父类的方法)
  - getDeclaredMethods()获取本类所有方法 

#### 3.1.3 反射场景

- 利用反射和配置文件，可以使应用程序更新时，对源码无需任何修改，便可发送新类给客户端
- 通过反射越过泛型检查

### 3.2 代理

#### 3.2.1 代理模式

- 代理（Proxy）模式使结构型的设计模式之一，它可以为其他对象提供一种代理（Proxy）以控制对这个对象的访问。
- 所谓代理，是指具有与被代理的对象具有相同的接口的类，客户端必须通过代理与被代理的目标类交互，而代理一般在交互的过程中（交互前后），进行某些特别的处理。
- 通俗的来讲代理模式就是我们生活中常见的中介或者代理人，比如我们想要买房子或者买车，自己弄太麻烦，就可以找一个中介，帮我全部打理好

#### 3.2.2 代理模式的作用

- **中介隔离作用：**  在某些情况下，一个客户类不想或者不能直接引用一个委托对象，而代理对象可以在客户类和委托对象之间起到中介的作用，其特征是代理类和委托类实现相同的接口。
- **开闭原则，增加功能：**  代理类除了使客户类和委托类的中介之外，我们还可以通过给代理类对象增加额外的功能来扩展委托类的功能，这样做我们只需要修改委托类，符合代码设计的开闭原则。

#### 3.2.3 代理参与角色

- **抽象主题：**  真实主题与代理主题的共同接口。
- **真实主题：**  实现抽象主题，定义真实主题所要实现的业务逻辑，供代理主题调用
- **代理主题：**  实现抽象主题，是真实主题的代理。通过真实主题的业务逻辑方法来实现抽象方法，并可以附加自己的操作。

#### 3.2.4 应用场景

- 需要控制对目标对象的访问。
- 需要对目标对象进行进行方向增强。
- 需要延迟加载目标对象。

#### 3.2.5 CGlib动态代理

静态代理和proxy动态代理模式都是要求目标对象是实现一个接口的目标对象，但是有时候目标对象知识一个单独的对象，并没有实现任何的接口，这个时候就可以使用以目标对象子类的方式类实现代理，这种方法就叫做：Cglib代理

- **过程**

Cglib代理，也叫做子类代理，他是在内存中构建一个子类对象从而实现对目标对象功能的扩展。CGLib采用了非常底层的字节码技术，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。但因为采用的是继承，所以不能对final修饰的类进行代理。JDK动态代理与CGLib与CGLib动态代理均是实现SpringAOP的基础。

#### 3.2.6 两种动态代理的区别

- JDK动态代理是实现了被代理对象的接口，Cglib是继承了被代理对象。
- JDK和Cglib都是在运行期生成字节码，JDK是直接写Class字节码，Cglib使用ASM框架写Class字节码，Cglib代理实现更复杂，生成代理类比JDK效率低。
- JDK调用方法，是通过反射机制调用，Cglib是同各国FastClass机制直接调用方法，Cglib执行效率更高。

## 四、线程池

- **降低资源消耗**  通过重复已创建的线程降低线程创建和销毁造成的消耗
- **提高响应速度**  任务可以不需要等到线程创建就能立刻执行
- **提高现成的可管理性**  师兄用线程池可以对线程进行统一的分配，调优和监控

### 4.1 线程池参数

**ThreadPoolExecutor**

```java
						  						int corePoolSize,    // 线程池核心线程个数
                          int maximumPoolSize,  // 线程池最大线程池个数
                          long keepAliveTime,   // 存活时间，闲置线程存活的最大时间
                          TimeUnit unit,  // 存活时间的时间单位
                          BlockingQueue<Runnable> workQueue,  // 等待执行的任务的阻塞队列
                          ThreadFactory threadFactory,  // 创建线程的工厂
                          RejectedExecutionHandler handler  // 拒绝策列，当队列满并且线程个数达到maximunPoolSize
```

### 4.2 线程池类型

- **newFixedThreadPool(int nThreads)**

  ```java
  new ThreadPoolExecutor(nThreads, nThreads,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>());
  ```

核心线程数和最大线程个数相同（无闲置线程），并且阻塞长度为Integer.MAX_VALUE。keepAliveTime=0说明只要线程个数比核心线程个数多并且当前空闲则回收。newFixedThreadPool(int nThreads, ThreadFactory)自定义线程创建工厂。

- **newSingleThreadExecutor()**

```java
new FinalizableDelegatedExecutorService
    (new ThreadPoolExecutor(1, 1,
                            0L, TimeUnit.MILLISECONDS,
                            new LinkedBlockingQueue<Runnable>()));
```

创建一个核心线程个数和最大线程个数都为1的线程池，其他同上。自定义线程工厂newSingleThreadExecutor(ThreadFactory threadFactory)

- **newCachedThreadPool()**

```java
			new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                              60L, TimeUnit.SECONDS,
                              new SynchronousQueue<Runnable>());
```

创建一个按需创建线程的线程池，初始线程个数为0，最多线程个数为Integer.MAX_VALUE，并且阻塞队列为同步队列。加入同步队列里的任务会马上执行，同步队列里面最多只有一个任务。自定义线程工厂newCachedThreadPool(ThreadFactory threadFactory)

### 4.3 源码分析

#### 4.3.1 execute 

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
    // step1: 核心线程个数小于corePoolSize，增加新线程
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // step2：先往任务队列中添加
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // step3:队列满就增加新线程，增加失败就拒绝
    else if (!addWorker(command, false))
        reject(command);
}
```

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    	// step1: cas操作增加线程数
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
		// step2：把并发安全的任务添加到workers里面，同时启动任务执行
        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
            w = new Worker(firstTask);
            final Thread t = w.thread;
            if (t != null) {
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

#### 4.3.2 工作线程Worker的执行

```java
Worker(Runnable firstTask) {
    // AQS state -> -1
    setState(-1); // inhibit interrupts until runWorker
    this.firstTask = firstTask;
    this.thread = getThreadFactory().newThread(this);
}
```

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    // AQS state -> 0
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

```java
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
        decrementWorkerCount();

    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        completedTaskCount += w.completedTasks;
        workers.remove(w);
    } finally {
        mainLock.unlock();
    }

    tryTerminate();

    // 如果当前线程个数小于核心个数，则增加
    int c = ctl.get();
    if (runStateLessThan(c, STOP)) {
        if (!completedAbruptly) {
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            if (min == 0 && ! workQueue.isEmpty())
                min = 1;
            if (workerCountOf(c) >= min)
                return; // replacement not needed
        }
        addWorker(null, false);
    }
}
```

#### 4.3.3 shutdown操作

```java
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        advanceRunState(SHUTDOWN);
        interruptIdleWorkers();
        onShutdown(); // hook for ScheduledThreadPoolExecutor
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
}
```

```java
private void advanceRunState(int targetState) {
    for (;;) {
        int c = ctl.get();
        if (runStateAtLeast(c, targetState) ||
            ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))))
            break;
    }
}
```

```java
private void interruptIdleWorkers(boolean onlyOne) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers) {
            Thread t = w.thread;
            if (!t.isInterrupted() && w.tryLock()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                } finally {
                    w.unlock();
                }
            }
            if (onlyOne)
                break;
        }
    } finally {
        mainLock.unlock();
    }
}
```

```java
final void tryTerminate() {
    for (;;) {
        int c = ctl.get();
        if (isRunning(c) ||
            runStateAtLeast(c, TIDYING) ||
            (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
            return;
        if (workerCountOf(c) != 0) { // Eligible to terminate
            interruptIdleWorkers(ONLY_ONE);
            return;
        }

        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    terminated();
                } finally {
                    ctl.set(ctlOf(TERMINATED, 0));
                    termination.signalAll();
                }
                return;
            }
        } finally {
            mainLock.unlock();
        }
        // else retry on failed CAS
    }
}
```

```java
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        advanceRunState(STOP);
        interruptWorkers();
        tasks = drainQueue();
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
    return tasks;
}
```

#### 4.4.4 shutdownNow

```java
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        advanceRunState(STOP);
        interruptWorkers();
        tasks = drainQueue();
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
    return tasks;
}
```

```java
private void interruptWorkers() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers)
            w.interruptIfStarted();
    } finally {
        mainLock.unlock();
    }
}
```

#### 4.4.5 awaitTermination

```java
public boolean awaitTermination(long timeout, TimeUnit unit)
    throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (;;) {
            if (runStateAtLeast(ctl.get(), TERMINATED))
                return true;
            if (nanos <= 0)
                return false;
            nanos = termination.awaitNanos(nanos);
        }
    } finally {
        mainLock.unlock();
    }
}
```

## 五、面向对象

### 5.1 多态

把派生类的对象类型动议看成它本身的基类，编译器在编译时期无法确定到底执行的是哪一个派生类的方法。程序会根据自身的类型执行恰当的代码。

- **早期绑定**  （编译器

面向过程语言编译器在编译的时候，在出现函数调用的时候，会产生度具体函数名字的引用。这样在程序运行的时候，执行到函数调用的语句，就会发现这里一个对具体函数方法的引用，就会把执行逻辑解析道这个具体函数方法的绝对地址上。

- **后期绑定**  （运行期

Java中使用一个特殊的代码来代替绝对调用。这段代码使用对象中存储的信息来计算方法主体的地址。因此每个对象根据特定代码位的内容而不同。

- **向上转型**

把子类当成其基类来处理的过程叫做“向上转型”。

### 5.2 抽象类和接口的区别

- 语法区别：

  - 抽象类可以有构造方法，接口中不能有构造方法。
  - 抽象类中可以有普通成员变量，接口中没有普通成员变量

  - 抽象类中可以包含非抽象的普通方法，接口中的所有方法必须都是抽象的，不能有非抽象的普通方法。

  - 抽象类中的抽象方法的访问类型可以是public，protected。但接口中的抽象方法只能是public类型的，并且默认即为public abstract类型。
  - 抽象类中可以包含静态方法，接口中不能包含静态方法
  - 抽象类和接口中都可以包含静态成员变量，抽象类中的静态成员变量的访问类型可以任意，但接口中定义的变量只能是public static final类型，并且默认即为public static final类型。
  - 一个类可以实现多个接口，但只能继承一个抽象类

- 应用上的区别：

  - 接口更多的是在系统架构设计方法发挥作用，主要用于定义模块之间的通信契约。
  - 而抽象类在代码实现方面发挥作用，可以实现代码的重用

## 六、锁

### 6.1 可见性问题的解决（synchronized和volatile）

- sychronized

某一个线程进入synchronized代码块前后，线程获得锁，并清空工作内存，从主内存拷贝共享变量最新的值到工作内存称为副本，执行代码，将修改后的副本的值刷新回主内存中，释放锁。

- volatile

如果一个线程对共享变量进行写回操作，那么其他所有线程对该变量的读取都要在主存中。

禁止指令重排序

**补充：** MESI缓存一致性和嗅探

- MESI缓存一致性

当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。

- 嗅探

每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。



### 6.2 原子性问题的解决

DK Atomic开头的原子类、synchronized、LOCK，可以解决原子性问题

**synchronized和ReentrantLock的区别：**

|               | synchronized | reentrantlock |
| ------------- | ------------ | ------------- |
| 底层          | jvm          | jdk，cas      |
| 释放          | 自动         | 手动          |
| 可中断        | 否           | 是            |
| 公平          | 否           | 可公平        |
| 绑定condition | 否           | 是            |



### 6.3 有序性问题

Happens-Before 规则如下：

- 程序次序规则：在一个线程内，按照程序控制流顺序，书写在前面的操作先行发生于书写在后面的操作
- 管程锁定规则：一个unlock操作先行发生于后面对同一个锁的lock操作
- volatile变量规则：对一个volatile变量的写操作先行发生于后面对这个变量的读操作
- 线程启动规则：Thread对象的start()方法先行发生于此线程的每一个动作
- 线程终止规则：线程中的所有操作都先行发生于对此线程的终止检测
- 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- 对象终结规则：一个对象的初始化完成(构造函数执行结束)先行发生于它的finalize()方法的开始

### 6.4 CAS

### 6.5 锁升级机制

**预备知识点：**

在 JVM 中，对象在内存中分为三块区域：

- 对象头

- - Mark Word（标记字段）：默认存储对象的HashCode，分代年龄和锁标志位信息。它会根据对象的状态复用自己的存储空间，也就是说在运行期间Mark Word里存储的数据会随着锁标志位的变化而变化。
  - Klass Point（类型指针）：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

- 实例数据

- - 这部分主要是存放类的数据信息，父类的信息。

- 对其填充

- - 由于虚拟机要求对象起始地址必须是8字节的整数倍，填充数据不是必须存在的，仅仅是为了字节对齐。

    Tip：不知道大家有没有被问过一个空对象占多少个字节？就是8个字节，是因为对齐填充的关系哈，不到8个字节对其填充会帮我们自动补齐。