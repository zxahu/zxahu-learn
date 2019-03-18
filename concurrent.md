
## 前言
挑选了若干章节，学习了并发编程的知识，针对学习的内容，做一些笔记以备忘。
1. Java并发机制的底层实现原理
2. 线程的简介、使用和实现原理
3. 锁的种类、使用和实现原理
4. 原子类 && 并发工具类
5. 线程池和Executor框架
6. 并发容器和框架

## 1. Java并发机制的底层实现原理
### Java对象头
对象头中的markWord部分，存储了hashCode和锁信息。64位虚拟机的markWord如下：   

| 锁状态 | 25bit | 31bit | 1bit | 4bit | 1bit(偏向锁) | 2bit(锁标志位) |
| --- |--- |--- |--- |--- |--- |--- |
| 无锁 | 没用到 | hashCode | | | 0 | 01 |
| 偏向锁 | threadId(54bit) Epoch(2bit) | | | | 1 | 01 |


### 锁的类型
1. 锁的状态（从低到高）：无锁状态 < 偏向锁状态 < 轻量级锁状态 < 重量级锁状态
2. 锁可以升级但不能降级

#### 1. 偏向锁
当线程访问同步快并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后线程进入和退出同步块时不需要进行CAS操作来加锁和解锁，只是简单测试下对象头里是否存着指向本线程的偏向锁。
1. 测试对象头的MarkWord中是否存储着指向当前线程的偏向锁
2. 如果存了，则表示已获取锁
3. 如果没存，则测试MarkWord的偏向锁标识是否为1
4. 如果是1， 则尝试使用CAS将对象头的偏向锁指向当前线程
5. 如果不是1， 则使用CAS竞争锁


#### 2. 轻量级锁
1. 线程在执行同步块之前，JVM会先在当前线程的栈帧中创建用于存储锁记录的空间，并将 要加锁的对象 头中的MarkWord复制到锁记录中。
2. 线程使用CAS将 要加锁的对象 头中的MarkWord 替换为指向锁记录的指针。
3. 如果成功，则获取锁成功；如果失败，则表示其他线程也在竞争锁，当前线程使用自旋来获取锁。如果自旋获取不到锁，修改markWord为重量级锁

#### 4. 重量级锁
TODO

#### 5. 自旋锁
线程在没有取得锁的时候，不被挂起，而转去执行一个空循环，（即所谓的自旋，就是自己执行空循环），若在若干个空循环后，线程如果可以获得锁，则继续执行。若线程依然不能获得锁，才会被挂起。

### Java原子操作的实现方式
使用锁和循环CAS的方式来实现院子操作
#### 1. CAS实现原子操作的三大问题
##### 1. ABA问题
解决办法：在锁名称前加版本号。   
示例：A -> B -> A  ==> 1A -> 2B -> 3A

##### 2. 循环时间长开销大
当CAS长时间不成功，会给CPU带来非常大的执行开销。      
解决办法：CPU提供pause指令。它的作用有：   
1. 延迟流水线执行指令，使CPU不过多消耗资源   
2. 避免在退出循环时因内存顺序冲突导致CPU流水线被清空   

##### 3. 只能保证一个共享变了的原子操作   
对多个共享变量进行操作时，没法使用CAS的方式保证原子性。解决办法：   
1. 使用锁   
2. 将多个共享变了合并成一个共享变量，例如，有变量i=2, j=a, 则可对ij=2a进行CAS    

## 2. 线程的简介、使用和实现原理
### 线程的状态
NEW : 初始状态，线程被构建但没有调用start()方法
RUNNABLE : 运行状态，包括 就绪 和 运行
BLOCKED : 阻塞状态，表示线程阻塞于锁
WAITING : 等待状态，表示需要等其他线程做一些特定的操作(通知或中断)
TIME_WAITING : 超时等待，在指定时间内自行返回
TERMINATED : 终止状态，表示线程执行完毕

```
NEW --> RUNNABLE // Thread.start()

RUNNABLE --> WAIT // Object.wait(), Object.join(), LockSupport.park()

RUNNABLE --> TIMED_WAITING // Thread.sleep(long), Object.wait(long), Thread.join(long), LockSupport.parkNanos(), LockSupport.parkUntil()

WAITING/TIMED_WATING --> RUNNABLE // Object.notify(), Object.notifyAll(), LockSupport.unpark(Thread)

RUNNABLE --> BLOCKED // 等待进入synchronized
```

### DEAMON线程
JAVA虚拟机中不存在DEAMON线程时，JAVA虚拟机就会退出

### 终止线程
interrept() 和cancel() : 是安全的中断方式，有机会去清理资源

### 线程间通信
#### 1. volatile 
volatile : 对于变量，每个线程都能拥有一份拷贝。
volatile修饰的变量，就是告知程序，任何对该变量的访问都要从共享内存中获取，任何对它的改变都要同步刷新回共享内存。

#### 2. synchronized
确保多个线程在同一时刻，只有一个线程处于synchronized修饰的方法或同步块中。

#### 3. synchronized的实现原理
1. 每个对象都有自己的监视器，监视器的获取是排他的。
2. 线程获取对象的监视器成功，则说明获取锁成功。
3. 如果获取监视器失败，则线程进入同步队列，线程状态变为BLOCKED。
4. 当线程释放了锁，释放操作会唤醒同步队列中的线程，它们会重新尝试获取监视器。

#### 4. ThreadLocal
threadLocal是线程变量，一个以threadLocal对象为键，任意对象为值的存储结构，这个结构会被附带在线程上。
实现原理：Thread对象有一个成员变量叫threadLocals, 它的类型是ThreadLocal.ThreadLocalMap 。这里维护了每个线程独有的key-value


## 3. 锁的种类、使用和实现原理
### Lock
#### AbstractQueueSynchronizer(队列同步器)
##### 1. 构成
是用来构建锁和其他同步组件的基础框架，使用了int成员变量来表示同步状态，内置FIFO队列来完成线程的排队工作。它主要由以下构成：
1. Node : 将当前线程以及等待状态等信息构造成一个节点，加入同步队列后，节点变为阻塞状态。当同步状态释放时，会把首节点唤醒，尝试再次获取同步状态。
2. 队列同步器：包含头节点 和 尾节点的引用。

AbstractQueuedSynchronizer实现了独占、共享、和超时获取的功能
##### 2. 独占式同步状态获取与释放
获取的逻辑：
1. 使用tryAcquire(int arg)来线程安全的获取同步状态，如果获取成功则表示已获取到锁
2. 如果获取失败，构造同步节点(Node.EXCLUSIVE), 加入同步队列的尾部
3. 使用acquireQueued(Node node, int arg)方法，使节点以死循环的方式来获取同步状态

特点：
1. tryAcquire方法是asbtract的，公平/非公平 是在这里实现的
2. 同步队列中，节点与节点之间是基本不互相通信的，只是判断自己的前驱是否为头节点，使得节点的释放规则基本符合FIFO

##### 3. 共享式同步状态获取与释放
```
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```
1. tryAcquireShared() 是第一次尝试获取同步状态，如果大于0，则表示获取成功
2. doAcquireShared() 是节点进入同步队列后，进入自旋状态，如果前驱为头节点，则尝试获取同步状态

### ReentrantLock
可重入锁：该锁能支持一个线程对资源重复加锁
#### 公平/非公平
1. 定义： 在绝对时间上，先对锁进行获取的请求，一定先被满足，那么这个锁是公平的。
2. 实现方式：在tryAcquire方法中，公平锁CAS之前会判断同步队列中是否有等待节点，如果没有才会去做CAS；而非公平锁，会直接cas   
3. 区别：非公平锁的线程切换开销更小

### ReentrantReadWriteLock
1. 读写锁：同一时刻可以允许多个读线程访问，但在写线程访问时，所有的读线程和其他写线程都被阻塞   
2. 维护读写锁的状态：int类型的状态字段，共32位，0-15位表示写状态，16-32位表示读状态
#### 写锁获取的步骤
1. 获取state，如果state = 0, 则发起CAS && 设置currentThread
2. 如果state != 0, 则获取写状态位，如果写状态位为0 且 当前拥有锁的线程非本线程，则表示已有读锁，返回失败

#### 读锁获取步骤
1. 如果其他线程已获取写锁，则当前线程获取读锁失败
2. 如果当前线程获取了写锁或写锁未被获取，则CAS增加读状态位(即16-32位的数字+1)

### LockSupport工具
LockSupport.park() : 阻塞当前线程，使用unpark() 或者 当前线程中断，才会返回   
LockSupport.parkNanos() : 阻塞当前线程，最长不超过指定时间。返回条件增加了超时返回   
LockSupport.unpark() : 唤醒处于阻塞状态的线程   

### Condition
TODO

## 5. 线程池和Executor框架

### 线程池

线程池由 核心线程池　和　工作队列　组成，运行方式如下：
1. 如果当前运行线程数小于corePoolSize, 则创建新线程来执行任务
2. 如果运行的线程数 >= corePoolSize, 则将任务加入BlockingQueue
3. 如果BlockingQueue已满，则创建新线程来执行任务
4. 如果创建新线程时，当前运行的线程数超过maximumPoolSize, 任务将被拒绝，并调用RejectedExecutionHandler.rejectExecution()方法处理

### 线程池配置
1. 如果CPU密集型任务，应配置尽可能小的线程，比如配N(cpu) + 1 个线程的线程池
2. 如果IO密集型任务，应配尽可能多的线程，如配2 * N(cpu) 个线程的线程池

## 6. 并发容器和框架
### 1. concurrentHashMap
#### 节点
##### Node
包含了key-value键值对，与hashMap中的定义相似。但对value和next属性设置了volatile同步锁，它不允许调用setValue方法直接改变Node的value域
##### TreeNode
树节点，当链表长度过长时，会转换成treeNode
##### TreeBin
包装treeNode节点，它替代了treeNode的根节点。concurrentHashMap的数组中，存放了TreeBin对象。
##### ForwardingNode
一个用于连接2个table的节点类，包含了nextTable指针，指向下一张表。

## 7. 问题
### 1. concurrentHashMap为什么get()方法不需要加锁
concurrentHashMap的节点是用node实现的，node的value和next字段是用volatile修饰的。根据volatile的原理，当另外一个线程尝试对value进行修改时，会导致get线程中的value值失效，即修改对get操作可见，保证get操作的线程去主存读值。

### 2. concurrentHashMap的table也有volatile修饰的作用是什么
保证在数组扩容的时候，数组对其他线程可见
