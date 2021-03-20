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

