## 多线程
### 问：你怎么理解多线程的

1. 定义：多线程是指从软件或者硬件上实现多个线程并发执行的技术。具有多线程能力的计算机因有硬件支持而能够在同一时间执行多于一个线程，进而提升整体处理性能。
2. 存在的原因：因为单线程处理能力低。打个比方，一个人去搬砖与几个人去搬砖，一个人只能同时搬一车，但是几个人可以同时一起搬多个车。
3. 实现：在Java里如何实现线程，Thread、Runnable、Callable。
4. 问题：线程可以获得更大的吞吐量，但是开销很大，线程栈空间的大小、切换线程需要的时间，所以用到线程池进行重复利用，当线程使用完毕之后就放回线程池，避免创建与销毁的开销。

### 线程同步/线程间通信的方式
https://fangjian0423.github.io/2016/04/18/java-synchronize-way/  
https://github.com/crossoverJie/Java-Interview/blob/master/MD/concurrent/thread-communication.md

### 锁
#### 锁是什么
锁是在不同线程竞争资源的情况下来分配不同线程执行方式的同步控制工具，只有线程获取到锁之后才能访问同步代码，否则等待其他线程使用结束后释放锁

#### [synchronized](https://mp.weixin.qq.com/s/0qyNS6wQUShhJHoMuyziMg)
通常和wait，notify，notifyAll一块使用。    
wait：释放占有的对象锁，释放CPU，进入等待队列只能通过notify/all继续该线程。  
sleep：则是释放CPU，但是不释放占有的对象锁，可以在sleep结束后自动继续该线程。  
notify：唤醒等待队列中的一个线程，使其获得锁进行访问。  
notifyAll：唤醒等待队列中等待该对象锁的全部线程，让其竞争去获得锁。

#### lock
拥有synchronize相同的语义，但是添加一些其他特性，如中断锁等候和定时锁等候，所以可以使用lock代替synchronize，但必须手动加锁释放锁

#### 两者的区别
* 性能：资源竞争激烈的情况下，lock性能会比synchronized好；如果竞争资源不激烈，两者的性能是差不多的
* 用法：synchronized可以用在代码块上，方法上。lock通过代码实现，有更精确的线程语义，但需要手动释放，还提供了多样化的同步，比如公平锁、有时间限制的同步、可以被中断的同步
* 原理：synchronized在JVM级别实现，会在生成的字节码中加上monitorenter和monitorexit，任何对象都有一个monitor与之相关联，当且一个monitor被持有之后，他将处于锁定状态。monitor是JVM的一个同步工具，synchronized还通过内存指令屏障来保证共享变量的可见性。lock使用AQS在代码级别实现，通过Unsafe.park调用操作系统内核进行阻塞
* 功能：比如ReentrantLock功能更强大
  1. ReentrantLock可以指定是公平锁还是非公平锁，而synchronized只能是非公平锁，所谓的公平锁就是先等待的线程先获得锁
  2. ReentrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程
  3. ReentrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制

**我们写同步的时候，优先考虑synchronized，如果有特殊需要，再进一步优化。ReentrantLock和Atomic如果用的不好，不仅不能提高性能，还可能带来灾难。**

### 有几种锁
#### 可重入锁 
如果锁具备可重入性，则称作为可重入锁。像synchronized和ReentrantLock都是可重入锁
#### 可中断锁 
可中断锁：顾名思义，就是可以interrupt()中断的锁。 在Java中，synchronized就不是可中断锁，而Lock是可中断锁。   
如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。 
#### 公平锁 
公平锁即尽量以请求锁的顺序来获取锁。比如同是有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该所，这种就是公平锁。 非公平锁即无法保证锁的获取是按照请求锁的顺序进行的。这样就可能导致某个或者一些线程永远获取不到锁。   
在Java中，synchronized就是非公平锁，它无法保证等待的线程获取锁的顺序。而对于ReentrantLock和ReentrantReadWriteLock，它默认情况下是非公平锁，但是可以设置为公平锁。
#### 读写锁 
读写锁将对一个资源（比如文件）的访问分成了2个锁，一个读锁和一个写锁。正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。
ReadWriteLock就是读写锁，它是一个接口，ReentrantReadWriteLock实现了这个接口。可以通过readLock()获取读锁，通过writeLock()获取写锁。

### volatile
功能：

1. 主内存和工作内存，直接与主内存产生交互，进行读写操作，保证可见性；
2. 禁止 JVM 进行的指令重排序。

### ThreadLocal
使用`ThreadLocal<UserInfo> userInfo = new ThreadLocal<UserInfo>()`的方式，让每个线程内部都会维护一个ThreadLocalMap，里边包含若干了 Entry（K-V 键值对），每次存取都会先的都当前线程，然后得到该线程对象中的Map，然后与Map交互。

### 线程池
#### 起源
new Thread弊端：

* 每次启动线程都需要new Thread新建对象与线程，性能差。线程池能重用存在的线程，减少对象创建、回收的开销。
* 线程缺乏统一管理，可以无限制的新建线程，导致OOM。线程池可以控制可以创建、执行的最大并发线程数。
* 缺少工程实践的一些高级的功能如定期执行、线程中断。线程池提供定期执行、并发数控制功能

[http://www.wangtianyi.top/blog/2018/05/08/javagao-bing-fa-wu-xian-cheng-chi/](http://www.wangtianyi.top/blog/2018/05/08/javagao-bing-fa-wu-xian-cheng-chi/)  
[https://www.jianshu.com/p/edd7cb4eafa0](https://www.jianshu.com/p/edd7cb4eafa0)

### 并发包工具类
[http://www.wangtianyi.top/blog/2018/05/01/javagao-bing-fa-xi-lie-si-:juc/](http://www.wangtianyi.top/blog/2018/05/01/javagao-bing-fa-xi-lie-si-:juc/)  
[https://blog.csdn.net/mzh1992/article/details/60957351](https://blog.csdn.net/mzh1992/article/details/60957351)