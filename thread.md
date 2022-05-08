进程:
    是系统运行程序的基本单位
线程:
    一个进程可以产生多个线程，多个线程共享进程的堆与方法区，每个线程都有自己的程序计数器，虚拟机栈，本地方法栈。
    程序计数器记录的是下一条指令的地址，主要目的是为了线程切换后，能够恢复到正确的执行位置。
    虚拟机栈:
        每个java方法执行的时候同时会创建一个栈帧用于存储局部变量表、操作数栈，常量池引用。
    本地方法栈:
        调用的都是虚拟机提供的native方法
    堆:
        主要存放新对象，几乎所有的对象都在这里分配内存
    方法区:
        主要用于存放已被加载的类信息，常量，静态变量等。。

    生命周期:
        new 初始状态
        runnable 运行状态 包含就绪和运行
        blocked 阻塞状态 表示线程被阻塞
        waiting 等待状态 表示线程进入等待 比如通知中断
        time_waiting 超时等待 
        terminated 终止状态

    上下文切换:
        保存当前线程的上下文，留待线程下次占用cpu的时候恢复现场，并记载下一个将要占用cpu的线程上下文

    sleep() wait() :
        sleep 没有释放锁
        wait 释放锁
        两者都能暂停线程的执行
        wait主要用于线程间的通信，sleep通常用于暂停
        wait需要notify唤醒
        wait用于同步代码块中
    
    start()/run():
        在new出一个thread 的时候线程处于new状态，调用start(),会启动一个线程并使线程进入就绪，等待时间片轮转到自己就可以进行运行，此时就会调用run().可以理解run()就是一个普通方法。


synchronized：
    主要解决的是多个线程之间访问资源同步性，保证被修饰的代码，只有一个线程执行。
    底层基于monitor（monitorenter ,monitorexit）锁，依赖于操作系统mutex lock，java的线程都是直接映射到操作系统原生线程之上，如果频繁操作的话，会从用户态切换到内核态，非常耗时



    使用方法:
        方法上:锁的是当前对象实例
            并没有monitor指令，使用的是ACC_SYNCHRONIZED标志,
        静态方法: 锁的是类的所有对象实例
        代码块: 制定加锁对象，对给定对象、类加锁
            在需要同步的代码前后加入monitorenter 和 monitorexit,执行monitoenter的时候，线程会尝试获取对象监视器monitor的持有权

        dcl：
    
    对象头: 
        markword Klasspoint构成；
        mark word用于存储对象自身的运行时数据，包含hashcode，分代年龄，锁标志位等。
        Klasspoint: 对象指向它的类元数据的指针，虚拟机通过这个指针要确定这个对象是那个类的实例

```java
        public class Singleton{
            private volatile static Singleton instance;
            private Singleton(){

            }

            public static Singleton getInstance(){
                if (instance == null){
                    synchronized (Singleton.class){
                        if (instance == null){
                            instance = new Singleton();
                        }
                    }
                }
                return instance;
            }
        }
```java
    volatile：目的是 在new Object() 其实是3步，1.分配内存空间 2.初始化对象 3.对象指向内存地址 123 ，132

    synchronized 锁的优化：
        偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化

        锁的四种状态:
            无锁
            偏向
            轻量级
            重量级

    synchronized和reentrantLock：
        都是可重入锁，当锁还没释放时，再次想要获取的时候，还是可以获取的，不过锁的计数器自增1
        前者是jvm层面，后者是依赖于api
        ReentrantLock 多了等待可中断、公平锁、可实现选择性通知

volatile：
    目的是为了解决内存缓存不一致问题
    防止jvm的指令重排序,
    jmm:
        不是从主存读取变量。而是将变量保存本地内存，导致一个线程在主存修改了一个变量的值，而另一个线程还在用本地内存的值，造成数据不一致。

并发编程三大特性:
    原子性:
    可见性:volatile
    有序性:volatile

ThreadLocal:    
    主要解决的就是让每个线程绑定自己的值，threadlocal存储的就是每个线程的私有数据
    数据结构:
        thread 包含 threadLocals inhertableThreadLocals
        ThreadLocalMap 拥有set/get方法
    ThreadLocalMap中的key为ThreadLocal的弱引用。而value是强引用，所以在threadLocal没有被强引用的情况下，在垃圾回收的时候，key会清理掉，而value不会。那么就会出现key为null的entry。那么就会造成内存泄漏，使用完的时候最好手动remove一下。

线程池:
    目的都是减少每次获取资源的消耗，提高对资源的利用率。

runnable 跟 callable区别
    runnable不会返回结果抛出异常，但是callable接口可以
execute() 和 submit()区别
    前者提交不需要返回值的任务、无法判断是否成功
    后者用于提交需要返回值的任务，返回一个future类型对象，通过future来判断是否成功，future.get()回去返回值，但是get()会阻塞当前线程，直到任务完成，可以使用get(long timeout,TimeUnit unit),来超时返回

创建线程池:
    1.不允许使用executors来创建，而是通过threadPoolExecutor来
    FixedThreadPool和singlethreadExecutor: 请求队列为MAX-VALUE,堆积大量请求造成oom，
    cachedthreadPool 和 scheduledThreadPool 运行创建线程数量为max-value,造成oom

    ThreadPoolExecutor:
        重要参数:
            corePoolSize: 核心线程数定义了最小可以同时运行的线程数量
            maximumPoolSize:  当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
            workQueue: 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。
            keepAliveTime：核心线程外的线程 多余corePoolSize，会等待时间，才会回收
            unit: 等待的时间单位
            threadFactory: 创建线程时用到的工厂
            handler:拒绝策略
                如果当前同时运行的线程数量达到最大线程数量并且队列也被方面任务时，就会执行拒绝
                    AbortPolicy：抛出拒绝任务处理异常
                    CallerRunsPolicy：调用执行自己的线程运行任务，直接调用execute的线程
                    DiscardPolicy：不处理任务，直接丢弃
                    DiscardOldestPolicy：丢弃最早未被处理的任务请求
    
    线程池运行机制:
        execute(runnable);
        1.判断当前线程池中执行的任务数量是否小于核心线程数，如果小于的话，就新增一个线程，并将任务加入到改线程
        2.如果当前执行任务>= 核心线程数，判断是否运行中，且队列可以加入任务，那么将任务加入队列
        3.如果线程池已满直接返回拒绝策略

        
Atomic类:
    可以不加锁解决并发问题，stampedReference解决ABA问题
```java
public final int get() //获取当前的值
public final int getAndSet(int newValue)//获取当前的值，并设置新的值
public final int getAndIncrement()//获取当前的值，并自增
public final int getAndDecrement() //获取当前的值，并自减
public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
public final void lazySet(int newValue)//最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。

``` 
    原理:  
        主要利用CAS + volatile和native方法来保证原子操作.避免synchronized的高开小
        CAS就是比较并交换
```java
// setup to use Unsafe.compareAndSwapInt for updates（更新操作时提供“比较并替换”的作用）
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;

```

AQS:
    如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源锁定，如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及唤醒时锁分配机制。
    AQS是用CLH队列锁实现，将暂时获取不到锁的线程加入到队列中
    CLH队列是一个虚拟的双向队列，即不存在队列实例，仅仅存在节点之间的关联关系。

    资源共享形式:
        1.独占exclusive
            reentrantlock 
                公平锁 按照线程在队列中的排队顺序，先到先得
                非公平锁 无视队列顺序，谁想抢到
        2.共享share 多个线程同时执行
            countdownlatch 倒计时器
            semaphore 允许多个线程同时访问
            cyclicBarrier (循环栅栏
            ReadWriteLock 
    设计模式是基于模板方法模式，自定义同步器时需要重写下面几个 AQS 提供的钩子方法：
        钩子方法: 一种被声明在抽象类的方法。模板设计模式通过钩子方法控制固定步骤的实现
```java
protected boolean tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
protected boolean tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
protected int tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
protected boolean tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
protected boolean isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。

```

CompletebleFuture：
    可以实现异步、串行、并行、或者等待所有线程执行完成等操作

cpu计算型 ：设置为N + 1
I/O密集型 : 设置为2N

阻塞队列可以通过加锁来实现，非阻塞队列可以通过 CAS 操作实现。

非阻塞:
    concurrentLinkedQueue
阻塞:
    ArrayBlockingQueue: 有界队列实现类，底层采用数组实现
    LinkedBlockingQueue: 单向列表实现的阻塞队列 可做有界、无界队列来使用，不指定大小是，大小为Integer.MAX_VALUE. 相比array,有更高的吞吐量
    priorityBlockingQueue: 优先无界队列