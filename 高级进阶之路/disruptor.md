##diruptor


Disruptor它是一个开源的并发框架，并获得2011 Duke’s 程序框架创新奖，*能够在无锁的情况下实现网络的Queue并发操作*。

官网介绍为 *A High Performance Inter-Thread Messaging Library*


1、核心对象

- Ring Buffer
*是一种用于表示一个固定尺寸、头尾相连的缓冲区的数据结构，适合缓存数据流。* RingBuffer 通常采用数组实现，对 CPU 缓存友好，性能比链表好。
。从3.0版本开始，其职责被简化为仅仅负责对通过 Disruptor 进行交换的数据（事件）进行存储和更新。在一些更高级的应用场景中，Ring Buffer 可以由用户的自定义实现来完全替代。
- Sequence 
通过顺序递增的序号来编号管理通过其进行交换的数据（事件），对数据(事件)的处理过程总是沿着序号逐个递增处理。

一个 Sequence 用于跟踪标识某个特定的事件处理者( RingBuffer/Consumer )的处理进度。

虽然一个 AtomicLong 也可以用于标识进度，但定义 Sequence 来负责该问题还有另一个目的，那就是防止不同的 Sequence 之间的CPU缓存伪共享(Flase Sharing)问题。
 采用缓存行填充的方式对long类型的一层包装，用以代表事件的序号。通过unsafe的cas方法从而避免了锁的开销；


- RingBuffer 
 基于数组的缓存实现，也是创建sequencer与定义WaitStrategy的入口；
- Sequencer 
生产者与缓存RingBuffer之间的桥梁。单生产者与多生产者分别对应于两个实现SingleProducerSequencer与MultiProducerSequencer。Sequencer用于向RingBuffer申请空间，使用publish方法通过waitStrategy通知所有在等待可消费事件的SequenceBarrier；

- Sequence Barrier
用于保持对RingBuffer的 main published Sequence 和Consumer依赖的其它Consumer的 Sequence 的引用。 Sequence Barrier 还定义了决定 Consumer 是否还有可处理的事件的逻辑。
消费者与缓存RingBuffer之间的桥梁。消费者并不直接访问RingBuffer，从而能减少RingBuffer上的并发冲突

- Wait Strategy
定义 Consumer 如何进行等待下一个事件的策略。 （注：Disruptor 定义了多种不同的策略，针对不同的场景，提供了不一样的性能表现）

- Event
消费事件。Event的具体实现由用户定义。它不是一个被 Disruptor 定义的特定类型，而是由 Disruptor 的使用者定义并指定。

- EventProcessor
EventProcessor 持有特定消费者(Consumer)的 Sequence，并提供用于调用事件处理实现的事件循环(Event Loop)。 事件处理器，是消费者线程池Executor的调度单元，是对事件处理EventHandler与异常处理ExceptionHandler等的一层封装

- EventHandler
Disruptor 定义的事件处理接口，由用户实现，用于处理事件，是 Consumer 的真正实现。

- Producer
即生产者，只是泛指调用 Disruptor 发布事件的用户代码，Disruptor 没有定义特定接口或类型。

- Disruptor
 Disruptor的使用入口。持有RingBuffer、消费者线程池Executor、消费者集合ConsumerRepository等引用。

 

      



```
RingBuffer——Disruptor底层数据结构实现，核心类，是线程间交换数据的中转地；
Sequencer——序号管理器，负责消费者/生产者各自序号、序号栅栏的管理和协调；
Sequence——序号，声明一个序号，用于跟踪ringbuffer中任务的变化和消费者的消费情况；
SequenceBarrier——序号栅栏，管理和协调生产者的游标序号和各个消费者的序号，确保生产者不会覆盖消费者未来得及处理的消息，确保存在依赖的消费者之间能够按照正确的顺序处理；
EventProcessor——事件处理器，监听RingBuffer的事件，并消费可用事件，从RingBuffer读取的事件会交由实际的生产者实现类来消费；它会一直侦听下一个可用的序号，直到该序号对应的事件已经准备好。
EventHandler——业务处理器，是实际消费者的接口，完成具体的业务逻辑实现，第三方实现该接口；代表着消费者。
Producer——生产者接口，第三方线程充当该角色，producer向RingBuffer写入事件。
```




2、主要的点

---

“分离竞争点问题”或者队列的“合并竞争点问题”：通过将所有的东西都赋予私有的序列号，并且只允许一个消费者写Entry对象中的变量来消除竞争，Disruptor 唯一需要处理访问冲突的地方，是多个生产者写入 Ring Buffer 的场景。

Disruptor相对于传统方式的优点：

没有竞争=没有锁=非常快。
所有访问者都记录自己的序号的实现方式，允许多个生产者与多个消费者共享相同的数据结构。
在每个对象中都能跟踪序列号（ring buffer，claim Strategy，生产者和消费者），加上神奇的cache line padding，就意味着没有为伪共享和非预期的竞争。

---

- 缓存行 

缓存行 是缓存中可以分配的最小存储单位

*为什么追加64字节能够提高并发编程的效率呢？* 因为对于英特尔酷睿i7，酷睿， Atom和NetBurst， Core Solo和Pentium M处理器的L1，L2或L3缓存的高速缓存行是64个字节宽，不支持部分填充缓存行，这意味着如果队列的头节点和尾节点都不足64字节的话，处理器会将它们都读到同一个高速缓存行中，在多处理器下每个处理器都会缓存同样的头尾节点，当一个处理器试图修改头接点时会将整个缓存行锁定，那么在缓存一致性机制的作用下，会导致其他处理器不能访问自己高速缓存中的尾节点，而队列的入队和出队操作是需要不停修改头接点和尾节点，所以在多处理器的情况下将会严重影响到队列的入队和出队效率。Doug lea使用追加到64字节的方式来填满高速缓冲区的缓存行，避免头接点和尾节点加载到同一个缓存行，使得头尾节点在修改时不会互相锁定。

*那么是不是在使用Volatile变量时都应该追加到64字节呢？* 不是的。在两种场景下不应该使用这种方式。第一：缓存行非64字节宽的处理器，如P6系列和奔腾处理器，它们的L1和L2高速缓存行是32个字节宽。第二：共享变量不会被频繁的写。因为使用追加字节的方式需要处理器读取更多的字节到高速缓冲区，这本身就会带来一定的性能消耗，共享变量如果不被频繁写的话，锁的几率也非常小，就没必要通过追加字节的方式来避免相互锁定。


- 伪共享False sharing
每个cpu核心或者线程都会可能往同一个缓存行写数据；并且对共享变量，同时cpu核心会有各自的缓存行

虽然两个线程修改各种独立变量，但是因为这些独立变量被放在同一个高速缓存区，性能就影响了

当多核CPU线程同时修改在同一个高速缓存行各自独立的变量时，会不自不觉地影响性能，这就发生了伪共享False sharing，伪共享是性能的无声杀手。

解决方便是*将高速缓存剩余的字节填充填满(pad)，确保不发生多个字段被挤入一个高速缓存区*


java8 提供 @sun.misc.Contented 来填充缓存行



对齐原理 https://blog.csdn.net/qq_22654611/article/details/52838622
缓存行  https://blog.csdn.net/opensure/article/details/46669337

---
    Disruptor 定义了 com.lmax.disruptor.WaitStrategy 接口用于抽象Consumer如何等待新事件，这是策略模式的应用。
       Disruptor 提供了多个WaitStrategy 的实现，每种策略都具有不同性能和优缺点，根据实际运行环境的 CPU 的硬件特点选择恰当的策略，并配合特定的 JVM 的配置参数，能够实现不同的性能提升。
       例如，BlockingWaitStrategy、SleepingWaitStrategy、YieldingWaitStrategy 等，其中，BlockingWaitStrategy 是最低效的策略，但其对CPU的消耗最小并且在各种不同部署环境中能提供更加一致的性能表现；SleepingWaitStrategy的性能表现跟BlockingWaitStrategy差不多，对CPU的消耗也类似，但其对生产者线程的影响最小，适合用于异步日志类似的场景；YieldingWaitStrategy 的性能是最好的，适合用于低延迟的系统。在要求极高性能且事件处理线数小于CPU逻辑核心数的场景中。
       • **BlockingWaitStrategy**: 对低延迟和吞吐率不像CPU占用那么重要
       • **BusySpinWaitStrategy**： CPU使用率高，低延迟
       • **LiteBlockingWaitStrategy**: BlockingWaitStrategy变种，实验性的
       • **PhasedBackoffWaitStrategy**： Spins, then yields, then waits，不过还是适合对低延迟和吞吐率不像CPU占用那么重要的情况
       • **SleepingWaitStrategy**: spin， then yield，然后sleep(LockSupport.parkNanos(1L))在性能和CPU占用率之间做了平衡。
       • **TimeoutBlockingWaitStrategy**： sleep一段时间。 低延迟。
       • **YieldingWaitStrategy**： 使用spin, Thread.yield()方式
 

http://huangyunbin.iteye.com/blog/1944386

--- 
###RingBuffer###







####1、创建生产者
创建有两种方式：multiple producer RingBuffer（多生产者）  和 single producer RingBuffer(单生产者)

**single producer RingBuffer(单生产者)**
```
 public static <E> RingBuffer<E> createSingleProducer(
        EventFactory<E> factory,
        int bufferSize,
        WaitStrategy waitStrategy)
    {
    
        SingleProducerSequencer sequencer = new SingleProducerSequencer(bufferSize, waitStrategy);

        return new RingBuffer<E>(factory, sequencer);
    }

//1、创建单个生产者Sequencer：SingleProducerSequencer-->SingleProducerSequencerFields（加了两个属性）-->SingleProducerSequencerPad-->AbstractSequencer
abstract class SingleProducerSequencerFields extends SingleProducerSequencerPad
{
    SingleProducerSequencerFields(int bufferSize, WaitStrategy waitStrategy)
    {
        super(bufferSize, waitStrategy);
    }

    /**
     * Set to -1 as sequence starting point
     */
    long nextValue = Sequence.INITIAL_VALUE;
    long cachedValue = Sequence.INITIAL_VALUE;
}
public AbstractSequencer(int bufferSize, WaitStrategy waitStrategy)
    {
        if (bufferSize < 1)
        {
            throw new IllegalArgumentException("bufferSize must not be less than 1");
        }
        //Integer.bitCount()方法用于统计二进制中1的个数
        if (Integer.bitCount(bufferSize) != 1)
        {
            throw new IllegalArgumentException("bufferSize must be a power of 2");
        }

        this.bufferSize = bufferSize;
        this.waitStrategy = waitStrategy;
    }
    
2、new RingBuffer<E>(factory, sequencer)-->

RingBuffer(
        EventFactory<E> eventFactory,
        Sequencer sequencer)
    {
        super(eventFactory, sequencer);
    }


RingBufferFields(
        EventFactory<E> eventFactory,
        Sequencer sequencer)
    {
        this.sequencer = sequencer;
        this.bufferSize = sequencer.getBufferSize();

        if (bufferSize < 1)
        {
            throw new IllegalArgumentException("bufferSize must not be less than 1");
        }
        if (Integer.bitCount(bufferSize) != 1)
        {
            throw new IllegalArgumentException("bufferSize must be a power of 2");
        }

        this.indexMask = bufferSize - 1; //默认bufferSize - 1，做与操作
        this.entries = new Object[sequencer.getBufferSize() + 2 * BUFFER_PAD]; //bufferSize + 2* BUFFER_PAD  ( BUFFER_PAD = 128 / scale;)
        fill(eventFactory);
    }
    //test scale=4 那么1024+2*128/2=1088
    private void fill(EventFactory<E> eventFactory)
    {
        for (int i = 0; i < bufferSize; i++)
        {
            //从BUFFER_PAD位置开始填充
            entries[BUFFER_PAD + i] = eventFactory.newInstance();
        }
    }
```

**multi producer RingBuffer(单生产者)**
```
 public static <E> RingBuffer<E> createMultiProducer(
        EventFactory<E> factory,
        int bufferSize,
        WaitStrategy waitStrategy)
    {
        MultiProducerSequencer sequencer = new MultiProducerSequencer(bufferSize, waitStrategy);

        return new RingBuffer<E>(factory, sequencer);
    }
    
    //创建MultiProducerSequencer
    public MultiProducerSequencer(int bufferSize, final WaitStrategy waitStrategy)
    {
        super(bufferSize, waitStrategy);
        availableBuffer = new int[bufferSize];
        indexMask = bufferSize - 1;
        indexShift = Util.log2(bufferSize); //log2操作
        initialiseAvailableBuffer();
    }
    //
     private void initialiseAvailableBuffer()
    {   //默认填充-1
        for (int i = availableBuffer.length - 1; i != 0; i--)
        {
            setAvailableBufferValue(i, -1);
        }

        setAvailableBufferValue(0, -1);
    }
    
    //
    private void setAvailableBufferValue(int index, int flag)
    {
        long bufferAddress = (index * SCALE) + BASE;
        // public native void putOrderedInt(Object obj, long offset, int value);
        //将obj对象的偏移量为offset的位置修改为value
        UNSAFE.putOrderedInt(availableBuffer, bufferAddress, flag);
    }
    
    private static final long BASE = UNSAFE.arrayBaseOffset(int[].class);
    private static final long SCALE = UNSAFE.arrayIndexScale(int[].class);
```
还有一种方式，其实后面的步骤是一样的
```
  public static <E> RingBuffer<E> create(
        ProducerType producerType,
        EventFactory<E> factory,
        int bufferSize,
        WaitStrategy waitStrategy)
    {
        switch (producerType)
        {
            case SINGLE:
                return createSingleProducer(factory, bufferSize, waitStrategy);
            case MULTI:
                return createMultiProducer(factory, bufferSize, waitStrategy);
            default:
                throw new IllegalStateException(producerType.toString());
        }
    }

```
2、SequenceBarrier 
https://blog.csdn.net/zhxdick/article/details/52148806
```
 /**
     * Wait for the given sequence to be available for consumption.
     *
     * @param sequence to wait for
     * @return the sequence up to which is available
     * @throws AlertException       if a status change has occurred for the Disruptor
     * @throws InterruptedException if the thread needs awaking on a condition variable.
     * @throws TimeoutException     if a timeout occurs while waiting for the supplied sequence.
     */
    long waitFor(long sequence) throws AlertException, InterruptedException, TimeoutException;

    /**
     * Get the current cursor value that can be read.
     *
     * @return value of the cursor for entries that have been published.
     */
    long getCursor();

    /**
     * The current alert status for the barrier.
     *
     * @return true if in alert otherwise false.
     */
    boolean isAlerted();

    /**
     * Alert the {@link EventProcessor}s of a status change and stay in this status until cleared.
     */
    void alert();

    /**
     * Clear the current alert status.
     */
    void clearAlert();

    /**
     * Check if an alert has been raised and throw an {@link AlertException} if it has.
     *
     * @throws AlertException if alert has been raised.
     */
    void checkAlert() throws AlertException;

```


--- 
sequence:提供了获取，设置，相加，CAS等操作
```

//填充，规避伪共享
class LhsPadding
{
    protected long p1, p2, p3, p4, p5, p6, p7;
}

class Value extends LhsPadding
{
    protected volatile long value;
}

class RhsPadding extends Value
{
    protected long p9, p10, p11, p12, p13, p14, p15;
}

public class Sequence extends RhsPadding
{
    //队列默认长度，long型
    static final long INITIAL_VALUE = -1L;
    private static final Unsafe UNSAFE;
    private static final long VALUE_OFFSET;

    static
    {
        UNSAFE = Util.getUnsafe();
        try
        {   //获取偏移
            VALUE_OFFSET = UNSAFE.objectFieldOffset(Value.class.getDeclaredField("value"));
        }
        catch (final Exception e)
        {
            throw new RuntimeException(e);
        }
    }
    //Unsafe. putOrderedObject，Unsafe. putOrderedInt，Unsafe. putOrderedLong这三个方法，
    //JDK会在执行这三个方法时插入StoreStore内存屏障，避免发生写操作重排序。
    //public native void putOrderedLong(Object obj, long offset, long value);==>设置obj对象中offset偏移地址对应的long型field的值为指定值。
    //这是一个有序或者有延迟的putLongVolatile方法，并且不保证值的改变被其他线程立即看到。
    //只有在field被volatile修饰并且期望被意外修改的时候使用才有用
    //初始值为-1L
    public Sequence(final long initialValue)
    {
        UNSAFE.putOrderedLong(this, VALUE_OFFSET, initialValue);
    }

    //返回sequence当前的值
    public long get()
    {
        return value;
    }
    //设置值，store-store屏障between this write and any previous store.
    public void set(final long value)
    {
        UNSAFE.putOrderedLong(this, VALUE_OFFSET, value);
    }

    /**
     * 设置值，
     * a Store/Store barrier between this write and any previous
     * write and a Store/Load barrier between this write and any
     * subsequent volatile read.*/
    //public native void putLongVolatile(Object obj, long offset, long value);==>设置obj对象中offset偏移地址对应的long型field的值为指定值
    public void setVolatile(final long value)
    {
        UNSAFE.putLongVolatile(this, VALUE_OFFSET, value);
    }

    public boolean compareAndSet(final long expectedValue, final long newValue)
    {
        //返回true or false
        return UNSAFE.compareAndSwapLong(this, VALUE_OFFSET, expectedValue, newValue);
    }

    public long incrementAndGet()
    {
        return addAndGet(1L);
    }

    /**
     * 加给定的值，返回加后的值
     */
    public long addAndGet(final long increment)
    {
        long currentValue;
        long newValue;

        do
        {
            currentValue = get();
            newValue = currentValue + increment;
        }
        while (!compareAndSet(currentValue, newValue));

        return newValue;
    }

    @Override
    public String toString()
    {
        return Long.toString(get());
    }
}



```

---

###启动###






--- 
#### long next(n)####
next(1) 最后还是用到了next(n)方法
MultiProducerSequencer和SingleProducerSequencer next方法的区别：MultiProducerSequencer多了一个CAS算法，获取消费者最小序列号的while循环和放到外面和CAS的while循环合并

**wrapPoint**是一个很关键的变量，这个变量决定生产者是否可以覆盖序列号nextSequence，wrapPoint=nextSequence - bufferSize；RingBuffer表现出来的是一个环形的数据结构，实际上是一个长度为bufferSize的数组
nextSequence - bufferSize如果nextSequence小于bufferSize,wrapPoint是负数，表示可以一直生产；如果nextSequence大于bufferSize ,wrapPoint是一个大于0的数，由于生产者和消费者的序列号差距不能超过bufferSize(超过bufferSize会覆盖消费者未消费的数据），wrapPoint要小于等于多个消费者线程中消费的最小的序列号，即cachedGatingSequence的值

```


```





















---

####扩展 ：UnSafe####

java不能直接访问操作系统底层，而是通过本地方法来访问。Unsafe类提供了硬件级别的原子操作，主要提供了以下功能：

1️⃣内存管理：allocateMemory、reallocateMemory、freeMemory分别用于分配内存，扩充内存和释放内存

2️⃣CAS操作：是通过compareAndSwapXXX方法实现的

3️⃣Unsafe.park()

4️⃣可以定位对象某字段的内存位置，也可以修改对象的字段值

5️⃣内存屏障 ***Fence


参考 [unsafe](https://blog.csdn.net/zdy0_2004/article/details/74580236)



--- 
https://blog.csdn.net/hustspy1990/article/details/78265476?locationNum=4&fps=1

https://www.cnblogs.com/daoqidelv/p/6995888.html





--- 
https://stephenmann.io/post/good-code-depends-on-good-names/ 待①
https://www.cnblogs.com/daoqidelv/p/6995888.html
https://blog.csdn.net/huantuo4908/article/details/70313181
3.0分析 https://www.cnblogs.com/daoqidelv/p/6995888.html
方法名解释  https://blog.csdn.net/zhxdick/article/details/52004864
 https://blog.csdn.net/zhxdick/article/category/6121943




