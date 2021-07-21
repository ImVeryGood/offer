# 一：Java基础
##线程池
[https://www.jianshu.com/p/7726c70cdc40](https://www.jianshu.com/p/7726c70cdc40 "线程池详解")
###什么是线程池
线程池就是创建若干个可执行的线程放入一个池（容器）中，有任务需要处理时，会提交到线程池中的任务队列，处理完之后线程并不会被销毁，而是仍然在线程池中等待下一个任务。

**一个线程池包括以下四个基本组成部分：**

- 线程池管理器（ThreadPool）：用于创建并管理线程池，包括 创建线程池，销毁线程池，添加新任务；
- 工作线程（PoolWorker）：线程池中线程，在没有任务时处于等待状态，可以循环的执行任务；
-  任务接口（Task）：每个任务必须实现的接口，以供工作线程调度任务的执行，它主要规定了任务的入口，任务执行完后的收尾工作，任务的执行状态等；
   -  任务队列（taskQueue）：用于存放没有处理的任务。提供一种缓冲机制。                                                            
   ###线程池的优势

- 降低系统资源消耗，通过重用已存在的线程，降低线程创建和销毁造成的消耗；
- 提高系统响应速度，当有任务到达时，通过复用已存在的线程，无需等待新线程的创建便能立即执行；
- 方便线程并发数的管控。因为线程若是无限制的创建，可能会导致内存占用过多而产生OOM，并且会造成cpu过度切换（cpu切换线程是有时间成本的（需要保持当前执行线程的现场，并恢复要执行线程的现场））。
- 提供更强大的功能，延时定时线程池
###线程池的主要参数
    public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
    }
1、corePoolSize（线程池基本大小）：当向线程池提交一个任务时，若线程池已创建的线程数小于corePoolSize，即便此时存在空闲线程，也会通过创建一个新线程来执行该任务，直到已创建的线程数大于或等于corePoolSize时，（除了利用提交新任务来创建和启动线程（按需构造），也可以通过 prestartCoreThread() 或 prestartAllCoreThreads() 方法来提前启动线程池中的基本线程。）

2、maximumPoolSize（线程池最大大小）：线程池所允许的最大线程个数。当队列满了，且已创建的线程数小于maximumPoolSize，则线程池会创建新的线程来执行任务。另外，对于无界队列，可忽略该参数。

3、keepAliveTime（线程存活保持时间）当线程池中线程数大于核心线程数时，线程的空闲时间如果超过线程存活时间，那么这个线程就会被销毁，直到线程池中的线程数小于等于核心线程数。

4、workQueue（任务队列）：用于传输和保存等待执行任务的阻塞队列。

5、threadFactory（线程工厂）：用于创建新线程。threadFactory创建的线程也是采用new Thread()方式，threadFactory创建的线程名都具有统一的风格：pool-m-thread-n（m为线程池的编号，n为线程池内的线程编号）。

###常见线程池
- newSingleThreadExecutor
单个线程的线程池，即线程池中每次只有一个线程工作，单线程串行执行任务
- newFixedThreadExecutor(n)
 固定数量的线程池，没提交一个任务就是一个线程，直到达到线程池的最大数量，然后后面进入等待队列直到前面的任务完成才继续执行
- newCacheThreadExecutor（推荐使用）
可缓存线程池， 当线程池大小超过了处理任务所需的线程，那么就会回收部分空闲（一般是60秒无执行）的线程，当有任务来时，又智能的添加新线程来执行。
- newScheduleThreadExecutor
大小无限制的线程池，支持定时和周期性的执行线程

## 装箱拆箱
装箱就是  自动将基本数据类型转换为包装器类型；拆箱就是  自动将包装器类型转换为基本数据类型

**面试：**

下面这段代码的输出结果是什么？

    public class Main {
    public static void main(String[] args) {
         
        Integer i1 = 100;
        Integer i2 = 100;
        Integer i3 = 200;
        Integer i4 = 200;
         
        System.out.println(i1==i2);输出true
        System.out.println(i3==i4); 输出false
    }
    }

为什么会出现这样的结果？输出结果表明i1和i2指向的是同一个对象，而i3和i4指向的是不同的对象。此时只需一看源码便知究竟，下面这段代码是Integer的valueOf方法的具体实现：

      public static Integer valueOf(int i) {
        if(i >= -128 && i <= IntegerCache.high)
            return IntegerCache.cache[i + 128];
        else
            return new Integer(i);
    }

而其中IntegerCache类的实现为：
         
        private static class IntegerCache {
        static final int high;
        static final Integer cache[];

        static {
            final int low = -128;

            // high value may be configured by property
            int h = 127;
            if (integerCacheHighPropValue != null) {
                // Use Long.decode here to avoid invoking methods that
                // require Integer's autoboxing cache to be initialized
                int i = Long.decode(integerCacheHighPropValue).intValue();
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - -low);
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
        }

        private IntegerCache() {}
    }



   从这2段代码可以看出，在通过valueOf方法创建Integer对象的时候，如果数值在[-128,127]之间，便返回指向IntegerCache.cache中已经存在的对象的引用；否则创建一个新的Integer对象。

　　上面的代码中i1和i2的数值为100，因此会直接从cache中取已经存在的对象，所以i1和i2指向的是同一个对象，而i3和i4则是分别指向不同的对象。

**谈谈Integer i = new Integer(xxx)和Integer i =xxx;这两种方式的区别。**

   第一种方式不会触发自动装箱的过程；而第二种方式会触发；

　　在执行效率和资源占用上的区别。第二种方式的执行效率和资源占用在一般性情况下要优于第一种情况（注意这并不是绝对的）。

##泛型

  **泛型：** 泛型，即“参数化类型”。一提到参数，最熟悉的就是定义方法时有形参，然后调用此方法时传递实参。那么参数化类型怎么理解呢？顾名思义，就是将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。
  
   **泛型的使用：**泛型有三种使用方式，分别为：泛型类、泛型接口、泛型方法
    

**泛型类实例：**

    //此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参 数常用于表示泛型
    //在实例化泛型类时，必须指定T的具体类型
    public class Generic<T>{ 
    //key这个成员变量的类型为T,T的类型由外部指定  
    private T key;

    public Generic(T key) { //泛型构造方法形参key的类型也为T，T的类型由外部指定
        this.key = key;
    }

    public T getKey(){ //泛型方法getKey的返回值类型为T，T的类型由外部指定
        return key;
    }
    }



      //泛型的类型参数只能是类类型（包括自定义类），不能是简单类型

      //传入的实参类型需与泛型的类型参数类型相同，即为Integer.

      Generic<Integer> genericInteger = new Generic<Integer>(123456);


      //传入的实参类型需与泛型的类型参数类型相同，即为String.

      Generic<String> genericString = new Generic<String>("key_vlaue");

      Log.d("泛型测试","key is " + genericInteger.getKey());

      Log.d("泛型测试","key is " + genericString.getKey());


**注意：**
- 
- 泛型的类型参数只能是类类型，不能是简单类型。
- 不能对确切的泛型类型使用instanceof操作。如下面的操作是非法的，编译时会出错。

**泛型接口：**


    //定义一个泛型接口
    public interface Generator<T> {
       public T next();

      }

**泛型方法：**


     `/**
     * 泛型方法的基本介绍
     * @param tClass 传入的泛型实参
     * @return T 返回值为T类型
     * 说明：
     *     1）public 与 返回值中间<T>非常重要，可以理解为声明此方法为泛型方法。
     *     2）只有声明了<T>的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法。
     *     3）<T>表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T。
     *     4）与泛型类的定义一样，此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型。
     */
    public <T> T genericMethod(Class<T> tClass)throws InstantiationException ,
      IllegalAccessException{
        T instance = tClass.newInstance();
        return instance;
     }`

## String,StrindBuilder,StringBuffer区别 ##


**java中对String的具体操作是：**
 
当String中的字符增加时，其实实际上是新开辟了一块存储空间存储改变后的字符串，然后将之前使用的那块空间通过GC进行回收


**String和StringBuilder的区别：**
由上可知：

String每次变化都会开辟新的存储空间，存储数据，然后将之前那块空间通过GC进行回收

StringBuilder是可变长度的，数据的增加是直接在现有空间内进行


**StringBuilder和StringBuffer的区别：**
如果你只是单纯的使用上述两个类，会发现他们的方法几乎相同，使用起来也区别不大

但是如果你在编译器中去查看StringBuffer和StringBuilder的源代码，你会发现StringBuffer的方法中几乎都使用了synchronized关键字，那么现在看来两者的区别就很明显了

StringBuffer    线程安全     多线程的应用程序中使用
StringBuilder   非线程安全  单线程的应用程序中使用


**String、StringBuilder和StringBuffer三者的运行速度比较：**

毫无疑问

StringBuilder-->StringBuffer-->String
String针对的是少量的字符串操作
大量字符串数据进行操作使用StringBuilder和StringBuffer

## Lock和synchronized的区别 ##


1. Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；

2. synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；
3. Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；
4. 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。 
5. Lock可以提高多个线程进行读操作的效率。
6. Lock只有代码块锁，synchronized有代码块锁和方法锁
7. Lock可以提高多个线程进行读操作的效率。

在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时（即有大量线程同时竞争），此时Lock的性能要远远优于synchronized。所以说，在具体使用时要根据适当情况选择。


**synchronized是java中的一个关键字，也就是说是Java语言内置的特性。那么为什么会出现Lock呢？**
我们了解到如果一个代码块被synchronized修饰了，当一个线程获取了对应的锁，并执行该代码块时，其他线程便只能一直等待，等待获取锁的线程释放锁，而这里获取锁的线程释放锁只会有两种情况：

1.获取锁的线程执行完了该代码块，然后线程释放对锁的占有；

2.线程执行发生异常，此时JVM会让线程自动释放锁。

　　那么如果这个获取锁的线程由于要等待IO或者其他原因（比如调用sleep方法）被阻塞了，但是又没有释放锁，其他线程便只能干巴巴地等待，试想一下，这多么影响程序执行效率。

因此就需要有一种机制可以不让等待的线程一直无期限地等待下去（比如只等待一定的时间或者能够响应中断），通过Lock就可以办到。
　　再举个例子：当有多个线程读写文件时，读操作和写操作会发生冲突现象，写操作和写操作会发生冲突现象，但是读操作和读操作不会发生冲突现象。

**但是采用synchronized关键字来实现同步的话，就会导致一个问题：**

如果多个线程都只是进行读操作，所以当一个线程在进行读操作时，其他线程只能等待无法进行读操作。

因此就需要一种机制来使得多个线程都只是进行读操作时，线程之间不会发生冲突，通过Lock就可以办到。

另外，通过Lock可以知道线程有没有成功获取到锁。这个是synchronized无法办到的。

总结一下，也就是说Lock提供了比synchronized更多的功能。但是要注意以下几点：

1）Lock不是Java语言内置的，synchronized是Java语言的关键字，因此是内置特性。Lock是一个类，通过这个类可以实现同步访问；

2）Lock和synchronized有一点非常大的不同，采用synchronized不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用；而Lock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。


**Lock锁的用法**

    public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
    }

####tryLock()
方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待。

####lockInterruptibly()
方法比较特殊，当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。也就使说，当两个线程同时通过lock.lockInterruptibly()想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。

    public void method() throws InterruptedException {
    lock.lockInterruptibly();
    try {  
     //.....
    }
    finally {
        lock.unlock();
    }  
    }

注意，当一个线程获取了锁之后，是不会被interrupt()方法中断的。因为本身在前面的文章中讲过单独调用interrupt()方法不能中断正在运行过程中的线程，只能中断阻塞过程中的线程。

因此当通过lockInterruptibly()方法获取某个锁时，如果不能获取到，只有进行等待的情况下，是可以响应中断的。

而用synchronized修饰的话，当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。

####使用ReentrantLock实现同步
ReentrantLock，意思是“可重入锁”

另外在ReentrantLock类中定义了很多方法，比如：

　　isFair()        //判断锁是否是公平锁

　　isLocked()    //判断锁是否被任何线程获取了

　　isHeldByCurrentThread()   //判断锁是否被当前线程获取了

　　hasQueuedThreads()   //判断是否有线程在等待该锁

　　在ReentrantReadWriteLock中也有类似的方法，同样也可以设置为公平锁和非公平锁。不过要记住，ReentrantReadWriteLock并未实现Lock接口，它实现的是ReadWriteLock接口。

lock()方法:上锁

unlock()方法：释放锁

     public void run() {
           while (flag){
            lock.lock();//加锁
            try {
                try {
                    buy();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            } finally {
                lock.unlock();//解锁
            }
 
        }

####ReadWriteLock
ReadWriteLock也是一个接口，在它里面只定义了两个方法：

    public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading.
     */
    Lock readLock();
 
    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing.
     */
    Lock writeLock();
    }

一个用来获取读锁，一个用来获取写锁。也就是说将文件的读写操作分开，分成2个锁来分配给线程，从而使得多个线程可以同时进行读操作。下面的ReentrantReadWriteLock实现了ReadWriteLock接口。




####.ReentrantReadWriteLock
如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁。

如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会一直等待释放写锁。
####可重入锁
如果锁具备可重入性，则称作为可重入锁。像synchronized和ReentrantLock都是可重入锁，可重入性在我看来实际上表明了锁的分配机制：基于线程的分配，而不是基于方法调用的分配。举个简单的例子，当一个线程执行到某个synchronized方法时，比如说method1，而在method1中会调用另外一个synchronized方法method2，此时线程不必重新去申请锁，而是可以直接执行方法method2。

    class MyClass {
    public synchronized void method1() {
        method2();
    }
     
    public synchronized void method2() {
         
    }
    }
上述代码中的两个方法method1和method2都用synchronized修饰了，假如某一时刻，线程A执行到了method1，此时线程A获取了这个对象的锁，而由于method2也是synchronized方法，假如synchronized不具备可重入性，此时线程A需要重新申请锁。但是这就会造成一个问题，因为线程A已经持有了该对象的锁，而又在申请获取该对象的锁，这样就会线程A一直等待永远不会获取到的锁。

而由于synchronized和Lock都具备可重入性，所以不会发生上述现象。
####可中断锁
可中断锁：顾名思义，就是可以相应中断的锁。

在Java中，synchronized就不是可中断锁，而Lock是可中断锁。

如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。

在前面演示lockInterruptibly()的用法时已经体现了Lock的可中断性。
####公平锁
公平锁即尽量以请求锁的顺序来获取锁。比如同是有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该所，这种就是公平锁。

非公平锁即无法保证锁的获取是按照请求锁的顺序进行的。这样就可能导致某个或者一些线程永远获取不到锁。

在Java中，synchronized就是非公平锁，它无法保证等待的线程获取锁的顺序。

而对于ReentrantLock和ReentrantReadWriteLock，它默认情况下是非公平锁，但是可以设置为公平锁。
####读写锁
读写锁将对一个资源（比如文件）的访问分成了2个锁，一个读锁和一个写锁。

正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。

ReadWriteLock就是读写锁，它是一个接口，ReentrantReadWriteLock实现了这个接口。

可以通过readLock()获取读锁，通过writeLock()获取写锁。

上面已经演示过了读写锁的使用方法，在此不再赘述。
### Synchronized

Synchronized是Java的关键字，也是Java的内置特性，在JVM层面实现了对临界资源的同步互斥访问，通过对对象的头文件来操作，从而达到加锁和释放锁的目的。使用Synchronized修饰的代码或方法，通常有如下特性：

- Synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生。
- 不能响应中断。
- 同一时刻不管是读还是写都只能有一个线程对共享资源操作，其他线程只能等待，性能不高。

正是因为上面的特性，所以Synchronized的缺点也是显而易见的：即如果一个代码块被synchronized修饰了，当一个线程获取了对应的锁，并执行该代码块时，其他线程便只能一直等待，因此效率很低。


###volatile

保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。并且volatile是禁止进行指令重排序。

所谓指令重排序，指的是处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。

volatile为了保证原子性，必须具备以下条件：

- 对变量的写操作不依赖于当前值
- 该变量没有包含在具有其他变量的不变式中


## ArrayList和LinkedList区别及使用场景 ##

LinkedList和ArrayList的差别主要来自于Array和LinkedList数据结构的不同。

1. ArrayList是基于数组实现的，LinkedList是基于双链表实现的。
另外LinkedList类不仅是List接口的实现类，可以根据索引来随机访问集合中的元素，
除此之外，LinkedList还实现了Deque接口，Deque接口是Queue接口的子接口，它代表一个双向队列，因此LinkedList可以作为双向队列 ，栈（可以参见Deque提供的接口方法）和List集合使用，功能强大。

2. 因为Array是基于索引(index)的数据结构，它使用索引在数组中搜索和读取数据是很快的，可以直接返回数组中index位置的元素，因此在随机访问集合元素上有较好的性能。Array获取数据的时间复杂度是O(1),但是要插入、删除数据却是开销很大的，因为这需要移动数组中插入位置之后的的所有元素。

3. 相对于ArrayList，LinkedList的随机访问集合元素时性能较差，因为需要在双向列表中找到要index的位置，再返回；但在插入，删除操作是更快的。因为LinkedList不像ArrayList一样，不需要改变数组的大小，也不需要在数组装满的时候要将所有的数据重新装入一个新的数组，这是ArrayList最坏的一种情况，时间复杂度是O(n)，而LinkedList中插入或删除的时间复杂度仅为O(1)。ArrayList在插入数据时还需要更新索引（除了插入数组的尾部）。

4. LinkedList需要更多的内存，因为ArrayList的每个索引的位置是实际的数据，而LinkedList中的每个节点中存储的是实际的数据和前后节点的位置。

**使用场景：**

1. 如果应用程序对数据有较多的随机访问，ArrayList对象要优于LinkedList对象；

2. 如果应用程序有更多的插入或者删除操作，较少的随机访问，LinkedList对象要优于ArrayList对象；
 
3. 不过ArrayList的插入，删除操作也不一定比LinkedList慢，如果在List靠近末尾的地方插入，那么ArrayList只需要移动较少的数据，而LinkedList则需要一直查找到列表尾部，反而耗费较多时间，这时ArrayList就比LinkedList要快。

## HashMap 和 HashTable 区别 ##

**HashMap 不是线程安全的**

HashMap 是 map 接口的实现类，是将键映射到值的对象，其中键和值都是对象，并且不能包含重复键，但可以包含重复值。HashMap 允许 null key 和 null value，而 HashTable 不允许。

**HashTable 是线程安全 Collection。**

HashMap 是 HashTable 的轻量级实现，他们都完成了Map 接口，主要区别在于 HashMap 允许 null key 和 null value,由于非线程安全，效率上可能高于 Hashtable。

**区别如下：**

- HashMap允许将 null 作为一个 entry 的 key 或者 value，而 Hashtable 不允许。
- HashMap 把 Hashtable 的 contains 方法去掉了，改成 containsValue 和 containsKey。因为 contains 方法容易让人引起误解。
- HashTable 继承自 Dictionary 类，而 HashMap 是 Java1.2 引进的 Map interface 的一个实现。
- HashTable 的方法是 Synchronize 的，而 HashMap 不是，在多个线程访问 Hashtable 时，不需要自己为它的方法实现同步，而 HashMap 就必须为之提供外同步。
- Hashtable 和 HashMap 采用的 hash/rehash 算法都大概一样，所以性能不会有很大的差异。



## continue、break、和 return 的区别？ ##

continue ：跳出当前循环，继续下一次循环。
break ：跳出整个循体，继续执行循环下面的语句。
return:
return：结束方法执行，用于没有返回值的方法
return value ：return 特定值，用于有返回值的方法


##  == 和 equals 区别？ ##
- ==：判断两个对象的地址是否相等（基本数据类型比较的是值，引用数据类型比较的是内存地址）
- equals()：判断两个对象内容是否相等，是 Object 里面的方法。


    public boolean equals(Object obj) {
      return (this == obj);
    }

## 对象的相等与指向他们的引用相等，两者有什么不同? ##

- 对象的相等：比内存中存放的内容是否相等
- 引用相等：内存地址是否相等

## 面向对象三大特征 ##

- 封装：隐藏对象的属性和实现细节，仅对外公开接口，控制在程序中属性的读和修改的访问级别，将抽象得到的数据和行为（或功能）相结合，形成一个有机的整体，也就是将数据与操作数据的源代码进行有机的结合，形成“类”，其中数据和函数都是类的成员。
- 继承：允许创建分等级层次的类。继承就是子类继承父类的特征和行为，使得子类对象（实例）具有父类的实例域和方法，或子类从父类继承方法，使得子类具有父类相同的行为。
- 多态：同一个行为具有多个不同表现形式或形态的能力。是指一个类实例（对象）的相同方法在不同情形有不同表现形式。多态机制使具有不同内部结构的对象可以共享相同的外部接口。这意味着，虽然针对不同对象的具体操作不同，但通过一个公共的类，它们（那些操作）可以通过相同的方式予以调用。

## 面向对象，关键字 abstract static interface

[https://blog.csdn.net/qq_37766026/article/details/90702387?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control](https://blog.csdn.net/qq_37766026/article/details/90702387?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control "抽象类和接口")

###抽象类
在面向对象的概念中，所有的对象都是通过类来描绘的，但是反过来，并不是所有的类都是用来描绘对象的，如果一个类中没有包含足够的信息来描绘一个具体的对象，这样的类就可以称之为抽象类。
####抽象方法：使用abstract修饰且没有方法体的方法。
特点：

① 抽象方法没有方法体，交给子类实现

② 抽象方法修饰符不能是private final static

③ 抽象方法必须定义在抽象类或者接口中

####抽象类：包含抽象方法的类，即使用abstract修饰的类。

特点：

① 抽象类不能被实例化，只能被继承

② 抽象类中可以不包含抽象方法（不包含抽象方法就没有太大意义，可以作为工具类防止被实例化）

③ 抽象类的子类可以不实现该类所有的抽象方法，但也必须作为抽象类（抽象派生类）

④ 抽象类的构造方法不能定义成私有（子类构造方法会调用父类构造方法）

⑤ 抽象类不能使用final修饰，final修饰的类不能被继承

相同点：

① 抽象类和接口都不能被实例化

② 抽象类和接口都可以定义抽象方法，子类/实现类必须覆写这些抽象方法

不同点：

① 抽象类有构造方法，接口没有构造方法

② 抽象类可以包含普通方法，接口中只能是public abstract修饰抽象方法（Java8之后可以）

③ 抽象类只能单继承，接口可以多继承

④ 抽象类可以定义各种类型的成员变量，接口中只能是public static final修饰的静态常量

###设计层面：

- 抽象类是对一种事物的抽象，即对类抽象，而接口是对行为的抽象。抽象类是对整个类整体进行抽象，包括属性、行为，但是接口却是对类局部（行为）进行抽象。
 
- 设计层面不同，抽象类作为很多子类的父类，它是一种模板式设计。而接口是一种行为规范，它是一种辐射式设计。

###抽象类和接口的使用场景

####抽象类的使用场景

- 既想约束子类具有共同的行为（但不再乎其如何实现），又想拥有缺省的方法，又能拥有实例变量
 
- 如：模板方法设计模式，模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中某些步骤的具体实现。

####接口的应用场景
① 约束多个实现类具有统一的行为，但是不在乎每个实现类如何具体实现

② 作为能够实现特定功能的标识存在，也可以是什么接口方法都没有的纯粹标识。

③ 实现类需要具备很多不同的功能，但各个功能之间可能没有任何联系。

④ 使用接口的引用调用具体实现类中实现的方法（多态）

## throdlocal

## socked

## TCp/Ip Http  https

## Jvm

## 反射

##集合框架
1. HashMap。几乎每家公司都问，主要是内部原理如hash算法、冲突解决方案、扩容方案、红黑树的优缺点等。必会的内容，不会就直接当场去世了。
1. HashSet。内部使用HashMap来实现，value设置为object。记住这个就好了。
1. ConcurrentHashMap。必问。他的并发原理以及好处，同时有些面试官也会问缺点等问题。
1. Hashtable、SychronizeMap。一般和ConcurrentHashMap一起问，进行对比。
1. CopyOnWriteArrayList。一般会作为线程安全方法来进行比较优缺点。
1. 集合框架重点还是在Map，但是其他的框架List和queue的原理也是要了解的。

##访问限制符
public protect default private 四个要懂，基础知识了。特别注意protect是可以跨包访问的。

##类
1. 4种内部类，特别注意每个class编译后都会产生一个class文件，不管静态或非静态。面试踩坑了
1. lambda的本质。就是匿名内部类。
1. 抽象类和接口的区别。这个很看理解，如果有开发过具体项目的会回答得更加深刻，这是背八股文体现不出来的。

##异常
1. 异常体系、分类、机制。
1. 与error的区别。

##IO
主要还是问NIO的原理以及优缺点。建议把缓冲流的原理也得学一学并进行比较

##线程池

1. 内部原理。必会的啊。
1. 关键参数作用及如何配置。重点在如何配置，需要结合具体的机器情况、任务情况等等考量。
1. 线程池的作用。不仅仅只是线程复用，更重要的是管理线程、控制线程数量。这个也比较考察具体的项目运用理解。
1. 常见的四种线程池。
##并发
1. sychronize。必问，java的锁机制。特别是jdk6之后的锁优化以及运用场景。为什么是重量级的、JVM层如何实现如果了解可以加分。
1. Lock。必问，AQS的原理最好懂。一般会拿来和synchronize比较。
1. volatile。必问，会拿来和锁比较，他的两个重要作用。更深点会问到cpu缓存一致性协议、以及指令重排的类型与原理。
1. CAS。必问，问原理以及ABA问题。
1. 死锁。一般询问如何解决或者产生的条件。
1. Object的wait和notify。阻塞唤醒，一般会用一个代码或者具体的场景来询问如何保证多线程同步。
1. ThreadLocal。原理、内存泄露等
1. 这一块问的还是比较多，而且大都可以深入去问，看自己的学习程度了。
##JVM
1. GC机制。必问。
1. 类加载机制。必问，同时还会问双亲委托机制。
1. 方法调用过程。这个也问的挺多，也看对JVM的学习程度了。
1. 线程与进程的内存关系。如一个线程占多少内存、一个进程可以开多少线程、一个进程占用多少内存等。
1. 内存分布。JMM、运行时数据区、native内存分布。很看对JVM的理解程度。

# Android #
**1.android不依赖具体activity弹出Dialog对话框，即全局性对话框.**

在创建Dialog时添加:

 `dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT);`

并在AndroidManifest.xml中添加:

    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />

## 附加： ##

**1.SDK环境配置（adb install apk环境配置）**

环境变量添加：

变量名：`ADB_HOME `

变量值：`D:\androidstudio\Android\Sdk\platform-tools`

变量名：`ANDROID_HOME `

变量值：`D:\androidstudio\Android\Sdk`

然后在变量名：`Path `下添加`;%ANDROID_HOME%;%ADB_HOME%;`

最后验证一下：打开cmd命令行窗口：分别输入 **adb  android **  两个命令进行验证，都没有出错，则配置成功。

**2.JDK 环境配置**

- 系统变量→新建 JAVA_HOME 变量 。变量值填写jdk的安装目录（例如 E:\Java\jdk1.7.0)。
- 系统变量→寻找 Path 变量→编辑在变量值最后输入 `%JAVA_HOME%\bin;`%JAVA_HOME%\jre\bin;（注意原来Path的变量值末尾有没有;号，如果没有，先输入；号再输入上面的代码）。
- 系统变量→新建 CLASSPATH 变量变量值填写   `.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar`（注意最前面有一点）。
- 检验是否配置成功 运行cmd 输入 java -version （java 和 -version 之间有空格）若如图所示 显示版本信息 则说明安装和配置成功。
**3.开发环境测试**

1.要把手机网络代理修改为发布系统的电脑IP
2.模拟器。电脑要添加对应电脑的Ip,`C:\Windows\System32\drivers\etc`添加对应Ip
例：
    
    ‘# Copyright (c) 1993-2009 Microsoft Corp.
     0.0.0.0 account.jetbrains.com
     10.3.0.200  m-dev.rongrun.com’
##【Android】四大组件归纳总结 
[https://www.cnblogs.com/y4ngyy/p/12496745.html](https://www.cnblogs.com/y4ngyy/p/12496745.html "四大组件")
四大组件均可使用android:process="name"在Manifest中声明成独立进程
Activity
生命周期
 ![](https://developer.android.google.cn/guide/components/images/activity_lifecycle.png?hl=zh-cn)

**4种启动模式**

Android使用回退栈来管理Activity实例。当前显示的Activity在栈顶，当点击后退或返回时，栈顶的Activity出栈。

可以指定Activity的启动模式来避免重复创建同一Activity

在AndroidManifest.xml中声明Activity的启动模式

    <activity android:name=".MyActivity"
          android:lauchMode="singleTask"></activity>

- standard
默认的启动模式，允许Activity被多次实例化，一个任务栈中会有多个Activity实例
- singleTop
处于栈顶的Activity会被重用，若不在栈顶则会被重新创建。重用时会调用原来实例的onNewIntent()函数
- singleTask(常用)
一个任务栈只允许存在一个Activity实例，当startActivity()时，若该Activity在栈内，则会将该Activity上的所有Activity销毁，使该Activity处于栈顶，并调用onNewIntent()方法
- singleInstance
一个Activity在独立的任务中开启，保证在系统中只有一个实例，所有的startActivity()都会重用该实例，并回调onNewIntent()方法

两个Activity互相切换时的生命周期

A：onCreate->onStart->onResume

这是在A中启动B活动，生命周期如下：

A: onPause

B: onCreate->onStart->onResume

A: onStop

从B中返回A活动时

B: onPause

A: onRestart->onStart->onResume

B: onStop->onDestroy

**Service**

https://blog.csdn.net/javazejian/article/details/52709857

当程序进入后台运行时，所需要做的操作可以通过Service实现。

在任何位置调用startService()启动服务。

每个服务只存在一个实例，每次调用startService()时会回调onStartCommand()；只需要调用一次stopService()或stopSelf()函数，服务会被停止。

普通Service运行在UI线程，若需要执行耗时操作需要新开线程。

生命周期#
onCreate()

onStartCommand(intent, flags, startId)

有三种返回值

- START_STICKY：当服务因内存不足被kill掉后，内存空闲时会尝试重建服务，重建成功则回调onStartCommand()，这是传入的intent为null.
- START_NOT_STICKY：当Service因内存不足而被系统kill后，即使系统内存再次空闲时，系统也不会尝试重新创建此Service.
- START_REDELIVER_INTENT：当Service因内存不足而被系统kill后，则会重建服务，并通过传递给服务的最后一个 Intent 调用 onStartCommand()，这个值适用于主动执行应该立即恢复的作业（例如下载文件）的服务.
- onDestroy()

调用stopService()或stopSelf()
IntentService
重写onHandleIntent()函数，在函数中完成耗时操作。IntentService会自动将操作执行在子线程中，并在完成时调用stopSelf()自我销毁
public class MyIntentService extends IntentService {
    @Override
    protected void onHandleIntent(Intent intent) {
        ...
    }
}
Binder(与服务连接)
当服务仅限本地应用使用，不需要跨进程工作，则可以实现自有的Binder类，让客户端通过该类直接访问服务中的公共方法。
首先需要创建ServiceConnection对象，代表与服务的连接，有两个方法
•	onServiceConnected(name, serivce)
系统会调用该方法传递服务的onBind()方法返回的IBinder, 通过该对象可以调用获取到Service的实例对象，进而调用服务端的公共方法。
•	onServiceDisconnected(name)
系统与服务意外中断时调用，unBind不会调用该方法
调用bindService(intent, ServiceConnection, flag)绑定相关服务，flag指绑定时是否自动创建Service，0表示不创建；BIND_AUTO_CREATE表示自动创建。
调用unbindService(ServiceConnection)
当最后一个客户端与服务取消绑定时，系统会将服务销毁
前台服务
•	startForeground(int id, Notification notification)
将当前服务设成前台服务，id参数为唯一标识通知的整型数，不得为0
•	stopForeground(boolean removeNotification)
Android8.0后需要开启前台服务要在Activity中startForegroundService(i)，且之后Service要在5s内调用startForeground()才能成功创建前台服务
如何保证Service不被杀死
•	内存资源不足
o	将onStartCommand()返回值设成START_STICKY或START_REDELIVER_INTENT，这样内存组后也会恢复服务
o	将服务设成前台服务，具备较高优先级
•	用户手动干预
如果不是force stop则会调用生命周期中的onDestroy()方法，可以在方法中发送广播重启服务。完备一些的话就启动两个服务，相互监听，相互重启。
Broadcast
https://www.jianshu.com/p/ca3d87a4cdf3
组成：发送广播的Broadcast，接受广播的BroadcastReceiver和传递消息的Intent。
类型：普通广播、有序广播、本地广播(LocalBroadcast)、Sticky广播
静态广播与动态广播
广播可分为静态注册和动态注册两种形式
•	静态注册
在Manifest.xml中声明静态广播
<receiver android:name=".MyReceiver">
    <intent-filter android:priority=1000>
        <action android:name="com.broadcast"
    </intent-filter>
</receiver>
•	动态注册
可以在onCreate的时候注册
MyBroadcastReceiver receiver = new MyBroadcastReceiver();
IntentFilter filter = new IntentFilter("my.action");
context.registerReceiver(receiver, filter);
在onDestroy的时候注销
unregisterReceiver(receiver);
•	静态广播与动态广播的区别
1.	静态广播在activity注销的时候也能够继续接收；动态广播在APP退出后就无法接收了
2.	动态广播在相同Priority下优先级比静态广播高
普通广播
异步广播，调用sendBroadcast(new Intent(ACTION))来发出广播
定义广播接收器
public class MyBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        ...
    }
}
在AndroidManifest.xml中注册:
<receiver android:name=".broadcast.MyBroadcastReceiver">
    <intent-filter>
        <action android:name=".."/>
    </intent-filter>
</receiver>
动态注册接收器:
registerReceiver(new MyBroadcastReceiver(), new IntentFilter(MY_ACTION));
有序广播
发送出去的广播被广播的接收者按照先后顺序接收
接收的顺序排序
•	按照Priority属性值从大到小
•	Priority相同则动态注册广播优先
本地广播
只限于应用的广播
使用LocalBroadcastManager.getInstance(context)来使用关于广播的操作函数:
•	registerReceiver(receiver, intentFilter)
•	unregisterReceiver(receiver)
•	sendBroadcast(new Intent(INTENT_NAME))
•	sendBroadcastSync(new Intent())
注册本地广播
mLocalBroadcastManager = LocalBroadcastManager.getInstance(this);
mReceiver = new MyBroadcastReceiver();
IntentFilter filter = new IntentFilter();
filter.addAction(ACTION_MY_TYPE);
mLocalBroadcastManager.registerReceiver(mReceiver,filter);
需要在onDestory()的中进行注销：
mLocalBroadcastManager.unregisterReceiver(mReceiver)
ContentProvider
ContentProvider可以将应用中的数据共享给其他应用访问，其他应用可以通过ContentProvider对应用中的数据进行增删改查。
也可以进行进程间数据的交互和共享，跨进程通信。




## 横竖屏切换时候 Activity 的生命周期
- 不设置 android:configChanges ，切屏会销毁当前 Activity，重新加载各个生命周期，切横屏时会执行一次，切竖屏时会执行两次。
- 设置 android:configChanges="orientation"
- 在Android5.1 即 API 23级别下，切屏会重新调用各个生命周期，切横、竖屏时只会执行一次。
- 在Android9 即API 28级别下，切屏不会重新调用各个生命周期，只会执行 onConfigurationChanged 方法
- 设置 android:configChanges="orientation|keyboardHidden|screenSize"：切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法。

##ActivityA 跳转 ActivityB 然后 B 按 back 返回 A，各自的生命周期顺序，A 与 B 均不透明？如果B是透明主题的又或是个DialogActivity 呢？
    A -> B
    A:onPause
    B:onCreate -> onStart -> onResume
    A:onStop
    
    B -> A
    B:onPause
    A:onRestart -> onStart -> onResume
    B:onStop -> onDestroy 

    //B是透明主题
    //A:不会调用 onStop   
    
##Android中进程的优先级？
- 前台进程：与用户正在交互的 Activity 或者 Activity 用到的 Service 等，如果系统内存不足时前台进程是最晚被杀死的。
- 可见进程：处于暂停状态(onPause)的 Activity 或者绑定在其上的 Service，即被用户可见，但由于失了焦点而不能与用户交互。
- 服务进程：其中运行着使用 startService 方法启动的 Service，虽然不被用户可见，但是却是用户关心的，例如用户正在非音乐界面听的音乐或者正在非下载页面下载的文件等；当系统要空间运行，前两者进程才会被终止。
- 后台进程：其中运行着执行 onStop 方法而停止的程序，但是却不是用户当前关心的，例如后台挂着的QQ，这时的进程系统一旦没了有内存就首先被杀死。
- 空进程：不包含任何应用程序的进程，这样的进程系统是一般不会让他存在的。
##Activty 和 Fragmengt 之间怎么通信，Fragmengt 和 Fragmengt怎么通信
- Handler
- 广播
- 事件总线：EventBus、RxBus、Otto
- 接口回调
- Bundle 和 setArguments(bundle)
- 
        // 步骤1：获取FragmentManager
    FragmentManager fragmentManager = getFragmentManager();

    // 步骤2：获取FragmentTransaction
    FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

    // 步骤3：创建需要添加的Fragment 
    final mFragment fragment = new mFragment();

    // 步骤4:创建Bundle对象
    // 作用:存储数据，并传递到Fragment中
    Bundle bundle = new Bundle();

    // 步骤5:往bundle中添加数据
    bundle.putString("message", "I love Google");

    // 步骤6:把数据设置到Fragment中
    fragment.setArguments(bundle);

    // 步骤7：动态添加fragment
    // 即将创建的fragment添加到Activity布局文件中定义的占位符中（FrameLayout）
    fragmentTransaction.add(R.id.fragment_container, fragment);
    fragmentTransaction.commit();

##Fragment->Fragment

     public String getMessage() {
    return “你好”;
    }

     Fragment1 cf = (Fragment1) getActivity()
                        .getFragmentManager().findFragmentById(R.id.content_fg);
      String msg = cf.getMessage();

##onSaveInstanceState()方法的作用 ? 何时会被调用？
- 系统配置发生改变时导致 Activity 被杀死并重新创建、资源内存不足导致优先级低的 Activity 被杀死。
 
- 系统会调用 onSaveInstanceState 来保存当前 Activity 的状态，此方法调用在onStop之前，与onPause 没有既定的时序关系；
 
- 当Activity被重建后，系统会调用 onRestoreInstanceState，并且把 onSaveInstanceState 方法所保存的 Bundle 对象同时传参给 onRestoreInstanceState 和 onCreate()，因此可以通过这两个方法判断Activity 是否被重建 ，调用在 onStart 之后；
**简单来说，当页面未经你同意意外杀死的时候回调用！**

##Activity的四种启动模式、应用场景 ？
- standard标准模式：每次启动一个 Activity 都会重新创建一个新的实例，不管这个实例是否已经存在，此模式的 Activity 默认会进入启动它的 Activity 所属的任务栈中；
 
- singleTop栈顶复用模式：如果新 Activity 已经位于任务栈的栈顶，那么此 Activity 不会被重新创建，同时会回调 onNewIntent 方法，如果新 Activity 实例已经存在但不在栈顶，那么 Activity 依然会被重新创建；
 
- singleTask栈内复用模式：只要 Activity 在一个任务栈中存在，那么多次启动此 Activity 都不会重新创建实例，并回调 onNewIntent 方法，此模式启动 Activity，系统首先会寻找是否 Activity 存在想要的任务栈，如果不存在，就会重新创建一个任务栈，然后把创建好 Activity 的实例放到栈中，具有 clearTop 功能；
 
- singleInstance单实例模式：这是一种加强的 singleTask 模式，具有此种模式的 Activity 只能单独地位于一个任务栈中，且此任务栈中只有唯一一个实例；
##什么是 ANR？ 如何避免？
应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应用程序无响应（ANR：Application Not Responding）对话框。用户可以选择让程序继续运行，但是，用户在使用应用程序时，并不希望每次都要处理这个对话框。因此，在程序里对响应性能的设计很重要，这样，系统不会显示 ANR 给用户。

不同的组件发生 ANR 的时间不同（均为前台）：
 
- Activity：5s
- BroadcastReceiver：10s
- Service：20s

开发机器上出现问题，可以通过查看 /data/anr/traces.txt 。

**出现原因：**

- 主线程做了耗时操作（IO操作等）
- 主线程使用 Object.wait()、Thread.sleep() 等错误的操作
- 应用在规定时间内没有处理完相关事件（Activity、BroadcastReceiver 、Service ）
**处理：**

- 使用 AsyncTask、Thread、HandlerThread 等处理耗时操作
- 使用 Handler 线程之间的通信，而不是使用 Object.wait() 或者 Thread.sleep() 来阻塞主线程
- onCreate 和 onResume 回调中尽量避免耗时的代码

##AsyncTask 的缺陷和问题，说说他的原理？

AsyncTask 是一种轻量级的异步任务类，可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并主线程中更新 UI，通过 AsyncTask 可以更加方便执行后台任务以及在主线程中访问 UI，但是 AsyncTask 并不适合进行特别耗时的后台任务，对于特别耗时的任务来说，建议使用线程池。

AsyncTask 是一个抽象的泛型类，提供了 Params（参数类型）、Progress（后台任务的执行进度和类型） 和 Result（后台任务的返回结果的类型）这三个泛型参数，如果 AsyncTask 不需要传递具体的参数，那么这三个泛型参数可以用 Void 来代替。
使用：

    public class LoadOnlyImage extends AsyncTask<String,Integer,Bitmap> {
	  private ImageView image = null;
	  public LoadOnlyImage(ImageView image){
		this.image = image;
	}
	@Override
	protected Bitmap doInBackground(String... params) {
		// TODO Auto-generated method stub
		Bitmap bitmap = null;
		 try {
             URL url = new URL(params[0]);
             HttpURLConnection conn = (HttpURLConnection) url.openConnection();
             conn.setRequestMethod("GET");
             conn.setConnectTimeout(1000); // 5秒
             if (conn.getResponseCode() == 200) {
               bitmap = BitmapFactory.decodeStream(conn.getInputStream());
             }
         } catch (IOException e) {
        	 bitmap = null;
         }
		 return bitmap;
	}
	
	@Override
	protected void onPostExecute(Bitmap result) {
		// TODO Auto-generated method stub
		if(result == null){
			image.setBackgroundResource(R.drawable.base_nodata);
		}
		image.setImageBitmap(result);
	}
    }
#### 线程池
线程池主要是用在并发操作中，在并发操作中线程比较多，如果每一个线程执行结束了就销毁，接下来如果新任务来都是时候就需要新建线程，这样频繁创建和销毁线程，会大大降低系统效率。
- 并发：同一个处理器在同一时间段处理多个任务，交替执行。
- 并行：不同处理器在同一时刻同时处理不同的任务。
AsyncTask 里面线程池是一个核心线程数为 CPU + 1，最大线程数为 CPU * 2 + 1，工作队列长度为128 的线程池，线程等待队列的最大等待数为 28，但是可以自定义线程池。线程池是由 AsyncTask 来处理的，线程池允许tasks并行运行，需要注意的是并发情况下数据的一致性问题，新数据可能会被老数据覆盖掉。所以希望tasks能够串行运行的话，使用SERIAL_EXECUTOR。

#### 原理（两个线程池 + Handler）

- AsyncTask 中有两个线程池（SerialExecutor 和 THREAD_POOL_EXECUTOR）和一个Handler（InternalHandler），SerialExecutor 用于任务的排队，THREAD_POOL_EXECUTOR 用于真正地执行任务，InternalHandler 用于将执行环境从线程池切换到主线程。
- sHandler 是一个静态的 Handler 对象，为了能够将执行环境切换到主线程，这就要求 sHandler这个对象必须在主线程创建。由于静态成员会在加载类的时候进行初始化，因此就要求AsyncTask 的类必须在主线程中加载，否则同一个进程中的 AsyncTask 都将无法正常工作。

#### 缺陷

* 生命周期：Activity 销毁之前，没有取消 AsyncTask，有可能让应用崩溃(crash)。
* 内存泄漏：如果 AsyncTask 被声明 非静态内部类，会保留一个对 Activity 的引用，AsyncTask 的后台线程还在执行，它将继续在内存里保留这个引用，导致 Activity 无法被回收，引起内存泄漏。
* 结果丢失：屏幕旋转或 Activity 在后台被系统杀掉等情况会导致 Activity 的重新创建，之前运行的 AsyncTask  会持有一个之前 Activity 的引用，这个引用已经无效，这时调用 onPostExecute()再去更新界面将不再生效。
* 并行还是串行
  * Android1.6 之前，串行
  * Android 1.6 之后，采用线程池处理并行任务
  * 从Android 3.0开始，一个线程来串行执行任务。可以使用 executeOnExecutor() 方法来并行地执行任务。

### 13 动画

- View 动画：
  - 作用对象是 View，可用 xml 定义，建议使用 xml 比较易读
  - 四种类型：平移、旋转、缩放、透明度
- 帧动画：
  - 通过 AnimationDrawable 实现，容易 OOM
- 属性动画：
  - 可作用于任何对象，可用 xml 定义，Android 3 引入，建议代码实现比较灵活
  - 包括 ObjectAnimator、ValuetAnimator、AnimatorSet
  - 时间插值器：根据时间流逝的百分比计算当前属性改变的百分比，系统预置匀速、加速、减速等插值器
  - 类型估值器：根据当前属性改变的百分比计算改变后的属性值，系统预置整型、浮点、色值等类型估值器
  - 使用注意事项：避免使用帧动画，容易OOM；界面销毁时停止动画，避免内存泄漏；开启硬件加速，提高动画流畅性
  - 硬件加速原理：将 cpu 一部分工作分担给 gpu ，使用 gpu 完成绘制工作；从工作分摊和绘制机制两个方面优化了绘制速度

#### 属性动画和 View 动画区别：

属性动画真正的实现了 view 的移动，补间动画对 view 的移动只是在不同地方绘制了一个影子，实际对象还是处于原来的地方。 当动画的 repeatCount 设置为无限循环时，如果在 Activity 退出时没有及时将动画停止，属性动画会导致Activity无法释放而导致内存泄漏，而补间动画却没问题。 xml 文件实现的补间动画，复用率极高。在 Activity切换，窗口弹出时等情景中有着很好的效果。 使用帧动画时需要注意，不要使用过多特别大的图，容导致内存不足。

#### 原理：

* 属性动画：其实就是利用**插值器和估值器**，来计算出各个时刻 View 的属性，然后通过改变View的属性来，实现View的动画效果。
* View 动画：只是影像变化，view的实际位置还在原来的地方。
* 帧动画：是在xml中定义好一系列图片之后，使用 AnimationDrawable 来播放的动画。
* 属性动画：
  - 插值器：根据时间的流逝的百分比来计算属性改变的百分比
  - 估值器：根据插值器的结果计算出属性变化了多少数值

### 14 Bundle 传递对象为什么需要序列化？

为 bundle 传递数据时，只支持基本数据类型，所以在传递数据时，要将对象序列化转化成可以存储或者可以传输的本质状态，即**字节流**。序列化后的对象可以在网络，页面之间传递，也可以存储到本地。

### 15 Android 各版本新版本

#### Android5.0

- **MaterialDesign设计风格**
- **支持 64 位ART虚拟机**（5.0推出的ART虚拟机，在5.0之前都是Dalvik。他们的区别是： Dalvik,每次运行,字节码都需要通过即时编译器转换成机器码(JIT)。 ART,第一次安装应用的时候,字节码就会预先编译成机器码(AOT)）
- 通知详情可以用户自己设计

#### Android6.0

- **动态权限管理**
- 支持快速充电的切换
- 支持文件夹拖拽应用
- 相机新增专业模式

#### Android7.0

- **多窗口支持**
- **V2签名**
- 增强的Java8语言模式
- 夜间模式

#### Android8.0（O）

- **优化通知**

  通知渠道 (Notification Channel) 通知标志 休眠 通知超时 通知设置 通知清除

- **画中画模式**：清单中 Activity 设置 android:supportsPictureInPicture

- **后台限制**

#### Android9.0（P）

- **室内WIFI定位**
- **“刘海”屏幕支持**
- 安全增强

#### Android10.0（Q）

- **夜间模式**：包括手机上的所有应用都可以为其设置暗黑模式。
- **桌面模式**：提供类似于PC的体验，但是远远不能代替PC。
- **屏幕录制**：通过长按“电源”菜单中的"屏幕快照"来开启。

#### Android11.0（R）

* 短信更新改进：聊天泡泡
* 隐私和权限：位置、麦克风和摄像头的一次性权限许可
* 内置屏幕录制
* 适配不同设备：折叠手机等
* **网络优化**：5G

### 16 Serialzable 和 Parcelable 的区别？

##### Serializable（Java自带）：

Serializable 是序列化的意思，表示将一个对象转换成存储或可传输的状态。序列化后的对象可以在网络上进传输，也可以存储到本地。

##### Parcelable（android专用）：

除了 Serializable 之外，使用 Parcelable 也可以实现相同的效果，不过不同于将对象进行序列化，Parcelable方式的实现原理是将一个完整的对象进行**分解**，而分解后的每一部分都是Intent所支持的数据类型，这也就实现传递对象的功能了。

作用：

* 永久性保存对象，保存对象的字节序列到本地文件中；

* 通过序列化对象在网络中传递对象；

* 通过序列化在进程间传递对象。

序列化方法的原则

* 在使用内存的时候，Parcelable 比 Serializable 性能高，所以推荐使用 Parcelable。

* Serializable 在序列化的时候会产生大量的临时变量，从而引起频繁的GC。

* Parcelable 不能使用在要将数据存储在磁盘上的情况，因为 Parcelable 不能很好的保证数据的持续性在外界有变化的情况下。尽管 Serializable 效率低点，但此时还是建议使用 Serializable 。

### 17 Json

JSON 的全称是 JavaScript Object Notation，也就是 JavaScript 对象表示法 JSON是存储和交换文本信息的语法，类似XML，但是比XML更小、更快，更易解析 JSON是轻量级的文本数据交换格式，独立于语言，具有可描述性，更易理解，对象可以包含多个名称/值对，比如：


`{"name":"test" , "age":25}`


### 18 Android 为每个应用程序分配的内存大小是多少？

不同手机型号，分配的内存大小不同，可以设置  android:largeheap = "true"。

### 19 Activity 的 startActivity 和 context 的 startActivity区别？

* 从 Activity 中启动新的 Activity 时，可以直接 mContext.startActivity(intent) 

*  Context 中启动 Activity 须给 intent 设置Flag:

`intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);`

### 20 怎么在Service中创建Dialog对话框？

`//设置类型
dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT)
//权限
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINOW" />`

### 21 程序A能否接收到程序B的广播？

可以，使用全局的 BroadCastRecevier 能进行跨进程通信，但是注意它只能被动接收广播，此外，LocalBroadCastRecevier 只限于本进程的广播间通信。

### 22 数据加载更多涉及到分页，你是怎么实现的？

一般 20 为一页，滑动到底部，监听滑动事件，加载更多。

### 23 编译期注解跟运行时注解有何不同？

编译期注解：RetentionPolicy.CLASS，作用域 class 字节码上，生命周期只有在编译器间有效，常使用 APT（注解处理器）+ JavaPoet（Java源代码生成框架），EventBus、Dagger 等使用。

运行时注解：RetentionPolicy.RUNTIME，注解不仅被保存到 class 文件中，jvm 加载 class 文件之后，仍然存在；获取注解，需要通过反射来获取运行时注解。

### 24 如何解析 xml？

#### SAX

**SAX(Simple API for XML)**：是一种**基于事件的解析器**，事件驱动的流式解析方式，从文件的开始顺序解析到文档的结束，不可暂停或倒退。 
**优点：**解析速度快，占用内存少。非常适合在 Android 移动设备中使用。 
**缺点：**不会记录标签的关系，而要让你的应用程序自己处理，这样就增加了你程序的负担。 
**工作原理：**对文档进行顺序扫描，当扫描到文档(document)开始与结束、元素(element)开始与结束、文档 (document)结束等地方时通知事件处理函数，由事件处理函数做相应动作，然后继续同样的扫描，直至文档结束。 

#### PULL

**PULL：**运行方式和 SAX 类似，都是**基于事件的模**式。不同的是，在 PULL 解析过程中返回的是**数字**，且我们需要自己获取产生的事件然后做相应的操作，而不像 SAX 那样由处理器触发一种事件的方法，执行我们的代码。 

**优点：** PULL解析器小巧轻便，解析速度快，简单易用，非常适合在Android移动设备中使用，Android系统内部在解析各种 XML 时也是用 PULL 解析器，**Android官方推荐开发者们使用Pull解析技术**。Pull解析技术是第三方开发的开源技术，它同样可以应用于 JavaSE 开发。

#### DOM

DOM：即对象文档模型，它是将整个XML文档载入内存(所以效率较低，不推荐使用)，每一个节点当做一个对象，结合代码分析。DOM实现时首先为XML文档的解析定义一组接口，解析器读入整个文档，然后构造一个驻留内存的树结构，这样代码就可以使用DOM接口来操作整个树结构。 由于DOM在内存中以树形结构存放，因此检索和更新效率会更高。**但是对于特别大的文档，解析和加载整个文档将会很耗资源。 当然，如果XML文件的内容比较小，采用DOM是可行的。** 
**工作原理：**使用DOM对XML文件进行操作时，首先要解析文件，将文件分为独立的元素、属性和注释等，然后以节点树的形式在内存中对XML文件进行表示，就可以通过节点树访问文档的内容，并根据需要修改文档。 

### 25 更新 UI 方式

- Activity.runOnUiThread(Runnable)
- Handler
- View.post(Runnable)，View.postDelay(Runnable, long)（可以理解为在当前操作视图UI线程添加队列）
- AsyncTask
- RxJava
- LiveData

### 26 jar 和 aar 的区别

* jar 包里面只有代码
* aar 里面有代码、资源文件，比如 drawable 文件，xml资源文件，对于一些不常变动的 Android Library，我们可以直接引用 aar，加快编译速度。

### 27 程序自启动？


`//AndroidManifest.xml
android:installLocation="internalOnly":表示程序只能被安装在内存中，如果内存为空，则程序将不能成功安装，因为安装在 SD 卡中时会接收不到系统的广播消息(暂时未验证)
    
//添加权限
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />    
//注册广播    
<!--注册接收系统开机广播消息的广播接收者-->
<receiver
    android:name=".broadcastReceiver.MyBroadcastReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
        <category android:name="android.intent.category.HOME" />
    </intent-filter>
</receiver>  
//高版本基本没用，每个手机厂商都有一个手机管家，可以在里面设置自启动管理      `     


### 28 BroadcastReceiver，LocalBroadcastReceiver 区别？

|          | BroadcastReceiver                                            | LocalBroadcastReceiver                       |
| -------- | ------------------------------------------------------------ | -------------------------------------------- |
| 应用场景 | 应用之间的传递消息                                           | 应用内部传递消息                             |
| 安全     | 使用 Content API，本质上它是跨应用的，所以在使用它时必须要考虑到不要被别的应用滥用 | 不需要考虑安全问题，因为它只在应用内部有效。 |
| 原理     | Binder                                                       | Handler                                      |

### 29 SharedPrefrences 的 apply 和 commit 有什么区别？

* **commit 返回 boolean**，表示修改是否提交成功，apply 没有返回值
* commit 将数据同步的提交到硬件磁盘，apply 先将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘

### 30 计算一个view的嵌套层级


`private int getParents(View view){
    if(view.getParents() == null) 
        return 0;
    } else {
      return (1 + getParents(view.getParents));
   }
}`

### 31 asset 目录与 res 目录的区别？

assets：不会在 R 文件中生成相应标记，存放到这里的资源在打包时会打包到程序安装包中。（通过 AssetManager 类访问这些文件）

res：会在 R 文件中生成 id 标记，资源在打包时如果使用到则打包到安装包中，未用到不会打入安装包中。

#Handler

[https://www.jianshu.com/p/b5b7d61c84ac](https://www.jianshu.com/p/b5b7d61c84ac "Handler常问问题")
![](../asset/handler.png)

#### 角色

* Handler : 负责发送并处理消息
* Looper：：负责关联线程以及消息的分发，在该线程下从 MessageQueue 获取 Message，分发给 Handler ；
* MessageQueue ：消息队列，负责消息的存储与管理，负责管理由 Handler 发送过来的 Message；
* Message：消息，封装信息；

#### 原理

- Handler 通过 sendMessage() 发送 Message 到消息队列 MessageQueue。
- Looper 通过 loop() 不断提取触发条件的Message，并将 Message 交给对应的 target handler（发送该消息的 Handler ）来处理。
- target handler调用自身的 handleMessage() 方法来处理 Message。
- 
    var handler = Handler {}
    handler.sendMessage(msg)
    handler.post(runnable) 

##### Handler 与 Looper 的关联


    //Handler 基本使用
    class LooperThread : Thread() {
       lateinit var mHandler: Handler
       override fun run() {
           Looper.prepare()//创建 Looper
           mHandler = Handler {
           }
           Looper.loop()//不断尝试从 MessageQueue 中获取 Message , 并分发给对应的 Handler
        }
    }

    //Handler 构造方法
    public Handler(Callback callback, boolean async) {
        ...
        mLooper = Looper.myLooper();
        //检查当前线程的 Looper 是否存在    
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread " + Thread.currentThread()
                        + " that has not called Looper.prepare()");
        }
        //Looper 持有一个 MessageQueue
        mQueue = mLooper.mQueue;
         ...
     }


Handler 跟线程的关联是靠 Looper 来实现的。

##### Message 的存储与管理

Handler post 等方法 最终都会调用 `MessageQueue.enqueueMessage(Message,long)`，所以 Message 由 MessageQueue 存储和管理。

##### Message 的分发与处理

Message  存储和管理之后，就要进行分发与处理，调用 Looper.loop() 。


    public static void loop() {
        ...
        final Looper me = myLooper();
        ...
        for (;;) {//死循环 不断从 MessageQueue 获取消息处理
            Message msg = queue.next(); 
            if (msg == null) {              
                return;
            }
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
            ...
            try {
                //msg.target 就是发送该消息的 Handler 
                msg.target.dispatchMessage(msg);
                dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            ...
            //回收 message    
            msg.recycleUnchecked();
        }
    }


    queue.next()：获取消息


    Message next() {
        ...
        for (;;) {
             //本地方法      
            nativePollOnce(ptr, nextPollTimeoutMillis);
            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {                   
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }
                if (mQuitting) {
                    dispose();
                    return null;
                }
        }
    }



    msg.target 是发送该消息的 Handler，回调该 Handler :


    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {//callback 优先级比较高
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }


#### 注意

##### 为什么能在主线程直接使用 Handler，而不需要创建 Looper？

ActivityThread.main() 方法中调用了 Looper.prepareMainLooper() 方法创建了 主线程的 Looper ,并且调用了 loop() 方法，所以我们就可以直接使用 Handler 了。

##### Handler 引起的内存泄露？

Handler 允许我们发送延时消息，如果在延时期间用户关闭了 Activity，那么该 Activity 会泄露。 这个泄露是因为 Message 会持有 Handler，而又因为 Java 的特性，内部类会持有外部类，使得 Activity 会被 Handler 持有，这样最终就导致 Activity 泄露。

解决：将 Handler 定义成静态的内部类，在内部持有 Activity 的弱引用，并在Acitivity的onDestroy()中调用handler.removeCallbacksAndMessages(null) 及时移除所有消息。
    private static class SafeHandler extends Handler {

    private WeakReference<HandlerActivity> ref;

    public SafeHandler(HandlerActivity activity) {
        this.ref = new WeakReference(activity);
    }

    @Override
    public void handleMessage(final Message msg) {
        HandlerActivity activity = ref.get();
        if (activity != null) {
            activity.handleMessage(msg);
        }
    }
    }
并且再在 Activity.onDestroy() 前移除消息，加一层保障：

    @Override
    protected void onDestroy() {
    safeHandler.removeCallbacksAndMessages(null);
    super.onDestroy();
    }
这样双重保障，就能完全避免内存泄露了。

注意：单纯的在 onDestroy 移除消息并不保险，因为 onDestroy 并不一定执行
##### Handler 里藏着的 Callback 能干什么？

优先处理消息的权利。

##### 创建 Message 实例的最佳方式

- 通过 Message 的静态方法 Message.obtain()
- 通过 Handler 的公有方法 handler.obtainMessage()

##### 主线程的死循环一直运行是不是特别消耗CPU资源呢？

涉及到 Linux pipe/epoll 机制，简单说就是在主线程的 MessageQueue 没有消息时，便阻塞在loop的 queue.next() 中的 nativePollOnce() 方法里，此时主线程会释放 CPU 资源进入休眠状态，直到下个消息到达或者有事务发生，通过往 pipe 管道写端写入数据来唤醒主线程工作。这里采用的 epoll 机制，是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质是同步I/O，即读写是阻塞的。所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。

##### handler postDelay这个延迟是怎么实现的？
    //加上当前时间
     SystemClock.uptimeMillis() + delayMillis
     msg.when = when; 

handler.postDelay 并不是先等待一定的时间再放入到MessageQueue中，而是直接进入MessageQueue，以 MessageQueue 的时间顺序排列和唤醒的方式结合实现的。

##### 妙用 Looper 机制

- 将 Runnable post 到主线程执行；
- 利用 Looper 判断当前线程是否是主线程。

##### 主线程的 Looper 不允许退出

主线程不允许退出，退出就意味 APP 要挂。

##### 子线程里弹 Toast 

    new Thread(new Runnable() {
    @Override
     public void run() {
      Looper.prepare();
      Toast.makeText(HandlerActivity.this, "test", Toast.LENGTH_SHORT).show();
      Looper.loop();
      }
    }).start();


##pipe/epoll机制

[Handler 都没搞懂，拿什么去跳槽啊？](https://juejin.im/post/5c74b64a6fb9a049be5e22fc#heading-7)

### 33 硬件加速

使用 GPU 实现图形绘制，通过底层软件代码，将 CPU 不擅长的图形计算转换成 GPU 专用指令，由GPU完成。

- CPU更擅长复杂逻辑控制，而GPU得益于大量ALU和并行结构设计，更擅长数学运算。
- 页面由各种基础元素（DisplayList）构成，渲染时需要进行大量浮点运算。
- 硬件加速条件下，CPU用于控制复杂绘制逻辑、构建或更新DisplayList；GPU用于完成图形计算、渲染DisplayList。
- 硬件加速条件下，刷新界面尤其是播放动画时，CPU只重建或更新必要的DisplayList，进一步提高渲染效率。
- 实现同样效果，应尽量使用更简单的DisplayList，从而达到更好的性能（Shape代替Bitmap等）。

### 34 Android 中 HashMap 的优化

* SpareArray


    //避免了对key的自动装箱,两个数组来进行数据存储的，一个存储key，另外一个存储value,查找使用二分查找法
    private int[] mKeys;
    private Object[] mValues;


* ArrayMap：

* <key,value> 映射的数据结构，类似于 SpareArray

如果 key 类型为 int 建议使用 SpareArray，类似 LongSparseMap。

如果 key 类型为其他类型，则使用 ArrayMap。

注意：SparseArray、ArrayMap  在增加、查找、删除都需要使用 二分查找法，数据量大，效率会低。

### 35 Android 适配方案

* 布局适配
  * 布局自适应屏幕尺寸：使用相对布局（RelativeLayout），禁用绝对布局（AbsoluteLayout）
  * 根据屏幕的配置来加载相应的 UI 布局
    * 尺寸限定符
      * 例如有一个平板，可以使用：res/layout-large/main.xml ，只适合Android 3.2版本前
    * 最小宽度限定符：例如 layout-sw600dp 的最小宽度限定符，只适合Android 3.2版本后
    * 布局别名
    * 屏幕方向限定符
  * 使用 sp、dp
  * 屏幕尺寸和屏幕密度之间适配（以某一分辨率为基准，生成所有分辨率对应像素数列表）
* 布局组件适配：使得布局组件自适应屏幕尺寸：wrap_content、match_parent、weight 
* 图片资源适配：图片资源组件自适应屏幕尺寸 ：.9.png 、只切一套图 xhdpi
* 用户界面流程适配：根据屏幕的配置来加载相应的用户界流程
  * 确定当前布局
  * 根据当前布局做出响应
  * 重复使用其他活动中的片段
  * 处理屏幕配置变化

推荐：今日头条的适配方案：**动态更改 density**

    private static void adaptScreen(final Activity activity, final int sizeInPx, final boolean isVerticalSlide) {
        final DisplayMetrics systemDm = Resources.getSystem().getDisplayMetrics();
        final DisplayMetrics appDm = App.getAppContext().getResources().getDisplayMetrics();
        final DisplayMetrics activityDm = activity.getResources().getDisplayMetrics();
        if (isVerticalSlide) {
            activityDm.density = activityDm.widthPixels / (float) sizeInPx;
        } else {
            activityDm.density = activityDm.heightPixels / (float) sizeInPx;
        }
        activityDm.scaledDensity = activityDm.density * (systemDm.scaledDensity / systemDm.density);
        activityDm.densityDpi = (int) (160 * activityDm.density);
        appDm.density = activityDm.density;
        appDm.scaledDensity = activityDm.scaledDensity;
        appDm.densityDpi = activityDm.densityDpi;
    }

    
       
     px:像素   px = dp * (dpi / 160) 
        dp:设备独立像素
       dpi:像素密度 dpi = 根号(宽^2 + 高^2) / 屏幕尺寸(inch)
    
       //基本公式
       px = dp * density;
       density = dpi / 160;
       px = dp * (dpi / 160);    

    

//假如要使用第三方的UI界面的时候，重新设置为系统的density即可

    public static void cancelAdaptScreen(final Activity activity) {
        final DisplayMetrics systemDm = Resources.getSystem().getDisplayMetrics();
        final DisplayMetrics appDm = App.getAppContext().getResources().getDisplayMetrics();
        final DisplayMetrics activityDm = activity.getResources().getDisplayMetrics();
        activityDm.density = systemDm.density;
        activityDm.scaledDensity = systemDm.scaledDensity;
        activityDm.densityDpi = systemDm.densityDpi;
        appDm.density = systemDm.density;
        appDm.scaledDensity = systemDm.scaledDensity;
        appDm.densityDpi = systemDm.densityDpi;
    }
  


[Android 屏幕适配：最全面的解决方案](https://www.jianshu.com/p/ec5a1a30694b)

[Android屏幕适配全攻略(最权威的官方适配指导)](https://www.jianshu.com/p/f46f2874f52e)

### 36 显式 Intent 和隐式 Intent

显式 Intent，明确指出目标组件的名称


    bt.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent = new Intent(MainActivity.this,secondActivity.class);
                startActivity(intent);
            }
        });


隐式 Intent，没有明确指出目标组件名称，intent-filter

* 动作(Action)

* 类别`(Category ['kætɪg(ə)rɪ] )`

* 数据(Data )

     <activity android:name=".secondActivity">
            <intent-filter>
                <action android:name="com.example.aries.androidtest.ACTION_START"></action>
                <category android:name="android.intent.category.DEFAULT"></category>
            </intent-filter>
     </activity>

### 37 广播传输的数据是否有限制，是多少，为什么要限制？

广播传输的数据一般经过 IPC，Intent 在传递数据时是有大小限制的，大约限制在 **1MB** 之内，IPC 需要把数据从内核copy到进程中，每一个进程有一个接收内核数据的缓冲区，默认是1M；如果一次传递的数据超过限制，就会出现异常。

### 38 安卓签名机制？

* 应用程序升级，验证 APP 的**唯一性**，包名和签名都一致才能升级

* 应用程序模块化，可以模块化部署多个应用到一个进程，只要他们的签名都一样

* 代码或者数据共享，同一个签名有相同的权限，可以共享数据和代码

### 39 merge、ViewStub、include 的作用？

merge: 删减多余的层级，优化 UI，多用于替换 FrameLayout 或者当一个布局包含另一个

ViewStub: 按需加载，不会影响 UI 初始化时的性能

include：重用布局文件

### 40 ContentProvider 使用

内容提供器，调用 Context 的 **getContentResolver** 方法。

    Uri uri = Uri.parse("content:xxx/xxx");
    Cursor cursor = getContentResolver().query(uri,new String[]{"author","price"},
        "name=?",new String[]{"Java"},null);
    if(cursor.moveToFirst()){
    do{
        Log.d("MainActivity",cursor.getString(cursor.getColumnIndex("name")));
        Log.d("MainActivity",cursor.getString(cursor.getColumnIndex("price")));
    }while(cursor.moveToNext());
    }
    cursor.close()


### 41 Android 怎么加速启动 Activity？

* 优化布局，不要嵌套太多
* 主线程不要做太多的耗时工作

### 42 Json 解析方式的两种区别？

* JSONArray、JSONObject
* Gson

### 43 Fragment懒加载

* support 版本中利用 setUserVisibleHint 和  onHiddenChanged


`public class LazyLoadFragment extends Fragment {
    //判断是否已进行过加载，避免重复加载
    private boolean isLoad=false;
    //判断当前fragment是否可见
    private boolean isVisibleToUser = false;
    //判断当前fragment是否回调了resume
    private boolean isResume = false;
    private boolean isCallUserVisibleHint = false;

    @Override
    public void onResume() {
        super.onResume();
        isResume=true;
        if (!isCallUserVisibleHint) isVisibleToUser=!isHidden();
        lazyLoad();
    }

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        Log.i(TAG, “setUserVisibleHint: “+isVisibleToUser+”  “+FragmentD.this);
        this.isVisibleToUser=isVisibleToUser;
        isCallUserVisibleHint=true;
        lazyLoad();
    }

    @Override
    public void onHiddenChanged(boolean hidden) {
        super.onHiddenChanged(hidden);
        Log.i(TAG, "onHiddenChanged: "+hidden+" "+FragmentD.this);
        isVisibleToUser=!hidden;
        lazyLoad();
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        isLoad=false;
        isCallUserVisibleHint=false;
        isVisibleToUser=false;
        isResume=false;
    }

    private void lazyLoad() {
        if (!isLoad&&isVisibleToUser&&isResume){
            //懒加载。。。
            isLoad=true;
        }
    }`

* Androidx 版本。

 在 Androidx 在 FragmentTransaction 中增加了 setMaxLifecycle 方法来控制 Fragment 所能调用的最大的生命周期函数，该方法可以设置活跃状态下 Fragment 最大的状态，如果该 Fragment 超过了设置的最大状态，那么会强制将 Fragment 降级到正确的状态。

ViewPager+Fragment: 在 FragmentPagerAdapter 与 FragmentStatePagerAdapter 新增了含有 **behavior** 字段的构造函数。

    public class LazyLoadFragment extends Fragment {
    //判断是否已进行过加载，避免重复加载
    private boolean isLoad=false;

    @Override
    public void onResume() {
        super.onResume();
        if (!isLoad&& !isHidden()){
            lazyLoad();
            isLoad=true;
        }
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        isLoad=false;
    }

    private void lazyLoad() {
            //懒加载。。。
    }
}




[Android Fragment 懒加载](https://www.cnblogs.com/Robin132929/p/13819386.html)

### 44 Bitmap 使用时候注意什么？

* 要选择合适的图片规格（bitmap类型）：

```
ALPHA_8   每个像素占用1byte内存        
ARGB_4444 每个像素占用2byte内存       
ARGB_8888 每个像素占用4byte内存（默认）      
RGB_565 每个像素占用2byte内存
```

* 降低采样率。**BitmapFactory.Options** 参数 inSampleSize 的使用，先把 options.inJustDecodeBounds 设为 true，只是去读取图片的大小，在拿到图片的大小之后和要显示的大小做比较通过 calculateInSampleSize() 函数计算 inSampleSize 的具体值，得到值之后。options.inJustDecodeBounds 设为 false 读图片资源。

* 复用内存。通过软引用，复用内存块，不需要再重新给这个 bitmap 申请一块新的内存，避免了一次内存的分配和回收，从而改善了运行效率。

* 使用 recycle() 方法及时回收内存。

* 压缩图片

### 45 多进程场景遇见过么？

* 音乐播放器
* 大图查看
* 多模块应用

### 46 Bitmap 的 recycler()

 Android 有自己的垃圾回收机制，如果只使用了少量的图片，回收与否关系不大。可是若有大量的 bitmap 需要垃圾回收，那么垃圾回收的次数过于频繁，会造成系统资源负荷，这时候还是调用 recycler() 比较好。

通过源码可以了解到，加载Bitmap到内存里以后，是包含**两部分内存区域**的。简单的说，一部分是**Java部分**的，一部分是**C部分**的。这个Bitmap对象是由Java部分分配的，不用的时候系统就会自动回收了

但是那个对应的**C可用**的内存区域，虚拟机是不能直接回收的，这个只能调用底层的功能释放。所以需要调用recycle()方法来释放C部分的内存

bitmap.recycle()方法用于回收该Bitmap所占用的内存，接着将bitmap置空，最后使用System.gc()调用一下系统的垃圾回收器进行回收，调用System.gc()并不能保证立即开始进行回收过程，而只是为了加快回收的到来。

### 47 一张Bitmap所占内存以及内存占用的计算


`Bitamp 所占内存大小 = 宽度像素 *（inTargetDensity / inDensity）* 高度像素 *（inTargetDensity / inDensity）* 一个像素所占的内存字节大小 `


注：这里 inDensity 表示目标图片的dpi（放在哪个资源文件夹下），inTargetDensity表示目标屏幕的dpi，所以你可以发现inDensity和inTargetDensity会对Bitmap的宽高进行拉伸，进而改变Bitmap占用内存的大小。

在Bitmap里有两个获取内存占用大小的方法。 

- **getByteCount()**：API12 加入，代表存储 Bitmap 的像素需要的最少内存。
- **getAllocationByteCount()**：API19 加入，代表在内存中为 Bitmap 分配的内存大小，代替了 getByteCount() 方法。
- 在**不复用 Bitmap** 时，getByteCount() 和 getAllocationByteCount 返回的结果是一样的。在通过**复用 Bitmap** 来解码图片时，那么 getByteCount() 表示新解码图片占用内存的大 小，getAllocationByteCount() 表示被复用 Bitmap 真实占用的内存大小

### 48 数据库升级增加表和删除表都不涉及数据迁移，但是修改表涉及到对原有数据进行迁移,如何实现？

* 将现有表命名为临时表 
* 创建新表
* 将临时表的数据导入新表
* 删除临时表

### 49 Canvas.save() 跟 Canvas.restore()的区别

* Canvas.save()：来保存 Canvas 的状态。save 之后，可以调用 Canvas 的平移、放缩、旋转、错切、裁剪等操作。
* Canvas.restore()：用来恢复 Canvas 之前保存的状态。防止 save 后对 Canvas 执行的操作对后续的绘制有影响。

save 和 restore 要配对使用，**restore 次数 <= save 少**

### 50 为什么bindService可以跟Activity生命周期联动？

* bindService 方法执行时，**LoadedApk** 会记录 **ServiceConnection 信息**。

* Activity 执行 finish 方法时，会通过 LoadedApk 检查 Activity 是否存在未注销/解绑的 BroadcastReceiver 和 ServiceConnection，如果有，那么会通知 AMS 注销/解绑对应的 BroadcastReceiver 和 Service，并打印异常信息，告诉用户应该主动执行注销/解绑的操作。

### 51 自定义 view 效率高于xml定义吗？说明理由。

不一定，如果自定义 view 计算复杂，可能效率不一定高，但是一般情况下，自定义 view 效率高于 xml 定义。

* 少了解析 xml
* 自定义 View 减少了 ViewGroup 与 View之间的测量，包括父量子，子量自身，子在父中位置摆放，当子 view变化时,父的某些属性都会跟着变化

### 52 Gradle 配置多渠道打包

我们使用友盟多渠道打包。


    android {  
    productFlavors {
        xiaomi {}
        baidu {}
        wandoujia {}
        _360 {}        // 或“"360"{}”，数字需下划线开头或加上双引号
    }
    }


或者使用 360 加固，配置多渠道打包。

#### 53 Handler、Thread和HandlerThread的差别

* Handler：在 Android 中负责发送和处理消息，线程之间的消息通信。

* Thread：Java进程中执行运算的最小单位，亦即执行处理机调度的基本单位。某一进程中一路单独运行的程序。

* HandlerThread：
  * 继承自 **Thread**。
  * 内部实现了 **Looper** ，便于消息的分发。

### 54 广播的两种注册方式 ？

* **静态注册**（常驻广播）：在 AndroidManifest.xml 配置 <receive>
  * 常驻，不受任何组件的生命周期影响
  * 如果程序被关闭，如果有信息广播来，程序依旧会被系统调用
  * 需要时刻监听广播
* **动态注册**（非常驻广播）：Context.registerReceiver()
  * 非常驻，灵活，跟随组建的生命周期变化
  * 不需要特定时刻监听广播
  * 当用来注册的 Activity 关掉后，广播也就失效了。

### 55 ddms 和 traceView 的区别？

ddms：davik debug monitor service。简单的说 ddms 是一个**程序执行查看器**，在里面可以看见线程和堆栈等信息。

traceView： 程序性能分析器。traceview 是 ddms 中的一部分内容。

traceview 是 Android 平台特有的数据采集和分析工具，它主要用于分析 Android 中应用程序的 hotspot（瓶颈）。Traceview 本身只是一个数据分析工具，而数据的采集则需要使用 Android SDK 中的 Debug 类或者利用DDMS 工具。二者的用法如下：开发者在一些关键代码段开始前调用 Android SDK 中 Debug 类的 startMethodTracing 函数，并在关键代码段结束前调用 stopMethodTracing 函数。这两个函数运行过程中将采集运行时间内该应用所有线程（注意，只能是 Java线程） 的函数执行情况， 并将采集数据保存到/mnt/sdcard/下的一个文件中。 开发者然后需要利用 SDK 中的 Traceview工具来分析这些数据。

### 56 说说ContentProvider、ContentResolver、ContentObserver 之间的关系？

* ContentProvider **内容提供者**：对外提供数据，通过 ContentProvider 把应用中的数据共享给其他应用访问，其他应用可以通过ContentProvider 对你应用中的数据进行添删改查。

* ContentResolver **内容解析者**：按照一定规则访问内容提供者的数据（其实就是调用内容提供者自定义的接口来操作它的数据）。
* ContentObserver **内容监听器**：监听(捕捉)特定Uri引起的数据库的变化，继而做一些相应的处理，它类似于数据库技术中的触发器(Trigger)，当ContentObserver所观察的Uri发生变化时，便会触发它。

### 57  实现竖向的 TextView？TextView 文字描边效果？

* [竖向的 TextView](https://blog.csdn.net/wei1583812/article/details/56678110)
* 使用TextPaint绘制相同文字在TextView的底部，TextPaint的字显示要比原始的字大一些，这样看起来就像是有描边的文字。

### 58 Service 是否在 main thread 中执行, service 里面是否能执行耗时的操作?

默认情况，如果没有显示的指 service 所运行的进程, Service 和 activity 是运行在当前 app 所在进程的 main thread(**UI 主线程**)里面。

service 里面不能执行耗时的操作(网络请求,拷贝数据库,大文件)

### 59 简单说一下 Activity？

四大组件之一，用于与用户交互，一个界面对应一个 Activity。

### 60 如何保存 Activity 的状态或者 Activiy 重启怎么保存数据？

使用 **onSaveInstanceState()**


    override fun onSaveInstanceState(outState: Bundle) {
     //保存状态
     super.onSaveInstanceState(outState)
    }


### 61 Context、 Activity、Application 有什么区别？

* Activity 和 Application 都是Context的子类
* Context：上下文，管理上下文环境中各个参数和变量。
* Activity 维护一个 Acitivity 的生命周期，其对应的 Context 也只能访问该 Activity内的各种资源。
* Application 维护一个 Application 的生命周期。

### 62 Context 是 什 么 ？  一 个 应用有多少个 Context？
* 描述的是一个应用程序环境的信息，即上下文。

* 抽象(abstract class)类，Android 提供了该抽象类的具体实现类。

* 通过它我们可以获取应用程序的资源和类，也包括一些应用级别操作，例如：启动一个Activity，发送广播，接受Intent信息
* Context 实例个数 = Service 个数 + Activity 个数 + 1（Application对应的Context实例）

### 63 Activity 怎么和 Service 绑定，怎么在Activity 中启动自己对应的 Service？

bindService(Intent service, ServiceConnection conn, int flags)。

 Activity 中可以通过 **startService** 和 **bindService** 启动 Service。一般情况下如果想获取 Service 的服务对象那么通过 bindService（），比如音乐播放器，第三方支付等。如果仅仅只是为了开启一个后台任务那么可以使用 startService（）方法。

### 64 Service 的生命周期

*  非绑定模式
  * 当第一次调用 startService ：onCreate() -> onStartCommand()
  * 多次调用 startService：onStartCommand()
  * 关闭：onDestory()
* 绑定模式
  * 第一次调用 bindService：onCreate() -> onBind(）
  * 解除绑定： onUnbind() -> onDestory()。

![](../asset/servie生命周期.png)

### 65 简单说一下 IntentService

* IntentService 是 Service 的子类，创建独立的 **worker 线程**来处理所有的 Intent 请求；
* 请求处理完成后，IntentService 会自动停止，无需调用 stopSelf() 停止 Service； 

### 66 说说 Activity、Intent、Service 是什么关系？

Activity 和 Service 都是 Android 四大组件之一。都是 Context 类的子类 **ContextWrapper** 的子类。

* Activity 和 Service 都是 Context 类的子类ContextWrapper 的子类

* Activity 负责用户界面的显示和交互，Service 负责后台任务的处理
* Intent 传递数据，因此可以把 Intent 看作是通信使者

### 67 Service 和 Activity 在同一个线程吗？

对于同一 app 来说默认情况下在同一个线程中，也就是 MainThread （UI 线程）。

### 68  Service 里面可以弹吐司么？

可以，Service  是 Context 的子类。

### 69  加速启动Activity？

* 优化布局文件
* onCreate() 和 onReume() 减少复杂操作，可以利用多线程
* 减少主线程阻塞时间

### 70 说说Android 集合框架？

* SparseArray：key 为 int 类型，value 为 Object。key 的查找使用了二分的查找方式。
* ArrayMap：存放了两个数组。一个存放 hash，另外一个存放的是真正的数值。二分法查找和实时扩容机制，实现了一个有序的HashMap.

### 71 断点续传和下载

* 断点续传和断点下载都是用的 RandomAccessFile, seek() 具有移动指定的文件大小的位置的功能。
* 在 Http 请求中，加入请求头 Range，下载指定区间的文件数。

### 73 Fragment 在 ViewPager 里面的生命周期，滑动 ViewPager 的页面时 Fragment 的生命周期的变化。

### 74 Android中跨进程通讯的几种方式?

* Bundle
* 通过系统文件
* ContentProvider
* Messenger
* AIDL

### 75 将一个Activity设置成窗口的样式？


`android:theme="@android:style/Theme.Dialog"`


### 76 HandlerThread

* HandlerThread 本质是一个线程类，继承了 Thread；
* HandlerThread 有自己的内部 Looper 对象，可以进行looper循环；
* 通过获取 HandlerThread 的 Looper 对象传递给Handler对象，可以在 handleMessage方法中执行异步任务；
* 创建HandlerThread后必须先调用HandlerThread.start()方法，Thread会先调用run方法，创建Looper对象。

### 77 IntentService

其实就是 Service + HandlerThread。

* IntentService是继承自 Service 并处理异步请求的一个类，在 IntentService 内有一个工作线程来处理耗时操作。
* 当任务执行完后，IntentService 会自动停止，不需要我们去手动结束。
* 如果启动 IntentService 多次，那么每一个耗时操作会以工作队列的方式在 IntentService 的 **onHandleIntent** 回调方法中执行，依次去执行，使用**串行**的方式，执行完自动结束。

### 78 说说 scheme 跳转协议？

Android中的 Scheme 是一种**页面跳转协议**，和网站通过URL的形式访问一样，APP同样可以通过这种方式进行跳转，可以满足以下需求：

- 当应用接收到 Push，点击通知栏消息跳转到特定页面，比如商品详情等。
- 通过服务器下发的跳转路径，客户端可以根据路径跳转相应页面。
- 应用跳转到其他 APP 指定页面。
- H5页面点击锚点，APP端跳转具体页面。

### 79 简单说说 LinearLayout、FrameLayout、RelativeLayout 性能？

* RelativeLayout 会对子View做两次 measure
* 如果 LinearLayout 中有weight属性，则也需要进行两次 measure，但即便如此，应该仍然会比RelativeLayout的情况好一点。
* RelativeLayout 的 子 View 如果高度和RelativeLayout不同，会引发效率问题，当子View很复杂时，这个问题会更加严重。如果可以，尽量使用 padding 代替 margin。

### 80 如何导入外部数据库

* 把原数据库放到 res/raw
* 把数据库复制到 /data/data/com.（package name）/ 目录下

### 81 如何保证Service不被杀死？

- 利用onStartCommand方法中，返回START_STICKY
- 提高优先级：android:priority="1000"
- onDestroy 方法里重启 Service
- 使用 startForeground 将 service 放到前台状态，提升service进程优先级

### 82 说一说支付流程？

[客户端 Android 集成流程](https://opendocs.alipay.com/open/204/105296/)

* 在支付宝开放平台创建应用，添加功能并签约，配置密钥。

* 导入支付宝 SDK

### 83 说说 AndroidManifest.xml？
* 是整个应用的主配置清单文件，用于记录应用的相关配置信息，包括应用的包名、组件、权限等。
* 申明组件（四大组件）

### 84 为什么 bindService 可以跟Activity生命周期联动？

* bindService 方法执行时，LoadedApk 会记录 ServiceConnection 信息。

* Activity 执行 finish 方法时，会通过 LoadedApk 检查 Activity 是否存在未注销/解绑的 BroadcastReceiver 和 ServiceConnection，如果有，那么会通知 AMS 注销/解绑对应的 BroadcastReceiver 和 Service，并打印异常信息，告诉用户应该主动执行注销/解绑的操作。

### 85 oom 是否可以try catch ？

可以，在try语句中声明了很大的对象，导致OOM，但是不合理，谨慎使用。

### 86 如何绕过 9.0 限制

* **通过某种方式修改函数的执行流程** （**inline hook**）
* **以系统类的身份去反射**
  - 直接把我们自己变成系统类
  - 借助系统类去调用反射（通过 「元反射」来反射调用 `VMRuntime.setHiddenApiExemptions` 将要使用的隐藏 API 全部都豁免掉了）
  - 参考：https://github.com/tiann/FreeReflection

[Android9.0非SDK接口限制](Android9.0非SDK接口限制.md)

### 87 你是如何做单元测试的？

- 单元测试（Junit4、Mockito、PowerMockito、Robolectric）
- UI测试（Espresso、Example）
- 压力测试（Monkey）

[Android单元测试只看这一篇就够了 ](https://www.jianshu.com/p/aa51a3e007e2)

### 88 非UI线程可以更新UI吗?

可以，viewRootImp 的创建是在 Activity中 onResume 方法中创建的，我们可以单独开启了线程在 onCreate 里，它逃过了viewRootImp的创建，所以不会抛异常，但是线程一旦阻塞两秒了，viewRootImp 已经创建好了，所以能检查到。


    void checkThread() {
    if (mThread != Thread.currentThread()) {
    throw new CalledFromWrongThreadException(
    "Only the original thread that created a view hierarchy can touch its views.");
    }
    }


### 89  怎么控制另外一个进程的View显示？

RemoteViews

### 90 Android 程序运行时权限与文件系统权限

* 运行时权限 Dalvik ( android授权)
* 文件系统 linux 内核授权

### 91 SurfaceView、TextureView、SurfaceTexture、GLSurfaceView？

- SurfaceView：使用双缓冲机制，有自己的 surface，在一个独立的线程里绘制，Android7.0之前不能平移、缩放
- TextureView：持有 SurfaceTexture，将图像处理为 OpenGL 纹理更新到 HardwareLayer，必须开启硬件加速，Android5.0之前在主线程渲染，之后有独立的渲染线程，可以平移、旋转、缩放
- SurfaceTexture：将图像流转为 OpenGL 外部纹理，不直接显示
- GLSurfaceView：加入 EGL 管理，自带 GL 上下文和 GL 渲染线程

### 92 Scroller 原理？


    //1.创建一个Scroller对象，一般在View的构造器中创建
     mScroller = new Scroller(context);
    
    //2.重写 View 的 computeScroll()
    @Override
    public void computeScroll() {
      super.computeScroll();
      if (mScroller.computeScrollOffset()) {
    scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
    postInvalidate();
      }
    }
    
    //3.调用startScroll()方法，startX和startY为开始滚动的坐标点，dx和dy为对应的偏移量
    mScroller.startScroll (int startX, int startY, int dx, int dy);
    invalidate();


* 在 mScroller.startScroll() 中为滑动做初始化准备，比如：起始坐标，滑动的距离和方向以及持续时间(有默认值)，动画开始时间等。

* mScroller.computeScrollOffset() 根据当前已经消逝的时间来计算当前的坐标点。因为在mScroller.startScroll()中设置了动画时间，那么在computeScrollOffset()方法中依据已经消逝的时间就很容易得到当前时刻应该所处的位置并将其保存在变量 mCurrX 和 mCurrY 中，除此之外该方法还可判断动画是否已经结束。

### 93  RecyclerView 的性能优化

**1、数据处理与视图绑定分离**

RecyclerView 的 bindViewHolder 方法是在 UI 线程进行的，如果在该方法进行耗时操作，将会影响滑动的流畅性。

**2、数据优化**

分页加载数据；

对于新增或删除数据通过DiffUtil，来进行局部数据刷新；

**3、布局优化**

减少过度绘制。

减少布局层级，可以考虑使用自定义 View 来减少层级，或者更合理的设置布局来减少层级。

**4、减少xml文件inflate时间**

使用 即new View()的方式创建布局，因为 xml 文件包括：layout、drawable 的 xml，xml 文件 inflate 出 ItemView 是通过耗时的 IO 操作。

**5、减少 View 对象的创建**

一个稍微复杂的 Item 会包含大量的 View，而大量的 View 的创建也会消耗大量时间，所以要尽可能简化 ItemView；设计 ItemType 时，对多 ViewType 能够共用的部分尽量设计成自定义 View，减少 View 的构造和嵌套。

**6、设置高度固定**

如果item高度是固定的话，可以使用 **RecyclerView.setHasFixedSize(true)**，来避免requestLayout浪费资源。

**7、共用RecycledViewPool**

在嵌套RecyclerView中，如果子RecyclerView具有相同的adapter，那么可以设置RecyclerView.setRecycledViewPool(pool)来共用一个RecycledViewPool。

**Note**: 如果LayoutManager是LinearLayoutManager或其子类，需要手动开启这个特性：layout.setRecycleChildrenOnDetach(true)。

**8、RecyclerView数据预取**

RecyclerView25.1.0及以上版本增加了Prefetch功能。

用于嵌套RecyclerView获取最佳性能。

详细分析：[RecyclerView 数据预取](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fentry%2F58a3f4f62f301e0069908d8f)。

**9、加大RecyclerView的缓存**

用空间换时间，来提高滚动的流畅性。


    recyclerView.setItemViewCacheSize(20);
    recyclerView.setDrawingCacheEnabled(true);
    recyclerView.setDrawingCacheQuality(View.DRAWING_CACHE_QUALITY_HIGH);


**10、增加RecyclerView预留的额外空间**

额外空间：显示范围之外，应该额外缓存的空间


    new LinearLayoutManager(this{
    @Override
    protected int getExtraLayoutSpace(RecyclerView.Statestate){
    return size;
      }
    };
    

**11、减少ItemView监听器的创建**

对ItemView设置监听器，不要对每个item都创建一个监听器，而应该共用一个XxListener，然后根据ID来进行不同的操作，优化了对象的频繁创建带来的资源消耗。

**12、优化滑动操作**

设置 RecyclerView.addOnScrollListener() 来在滑动过程中停止加载的操作。

**13、刷新闪烁**

调用notifyDataSetChange时，适配器不知道整个数据集中的那些内容以及存在，再重新匹配ViewHolder时会发生闪烁。

设置adapter.setHasStableIds(true)，并重写getItemId()来给每个Item一个唯一的ID

**14、回收资源**

通过重写RecyclerView.onViewRecycled(holder)来回收资源。

[RecyclerView 性能优化](https://www.jianshu.com/p/1853ff1e8de6)

### 94 ListView 与 RecyclerView 简单对比?

**缓存区别：**

- 层级不同：ListView有两级缓存，在屏幕与非屏幕内。RecyclerView比ListView多两级缓存，支持多个离屏ItemView缓存（匹配pos获取目标位置的缓存，如果匹配则无需再次bindView），支持开发者自定义缓存处理逻辑，支持所有RecyclerView共用同一个RecyclerViewPool(缓存池)。
- 缓存不同：ListView缓存View。RecyclerView缓存RecyclerView.ViewHolder，抽象可理解为：
  View + ViewHolder(避免每次createView时调用findViewById) + flag(标识状态)；

**优点**
RecylerView提供了局部刷新的接口，通过局部刷新，就能避免调用许多无用的bindView。
RecyclerView的扩展性更强大（LayoutManager、ItemDecoration等）。

### 95 Android类加载器

Android 中常用的有两种类加载器：

* DexClassLoader

* PathClassLoader

  它们都继承于 BaseDexClassLoader。区别在于调用父类构造器时，DexClassLoader 多传了一个optimizedDirectory 参数，这个目录必须是内部存储路径，用来缓存系统创建的 Dex 文件。而PathClassLoader该参数为null，只能加载内部存储目录的 Dex 文件。所以我们可以用 DexClassLoader 去加载外部的apk。

### 96 onStart()与onResume()有什么区别？

* `onStart()` 是 `Activity` 界面被显示出来的时候执行的，但不能与它交互；
* `onResume()` 是 当该 `Activity` 与用户能进行交互时被执行，用户可以获得它的焦点，能够与其交互。

### 97  Looper.loop()为什么不会阻塞主线程？

Android是基于事件驱动的，即所有Activity的生命周期都是通过Handler事件驱动的。loop方法中会调用MessageQueue的next方法获取下一个message，当没有消息时，基于 Linux pipe/epoll 机制会阻塞在loop的queue.next() 中的 nativePollOnce() 方法里，并不会消耗CPU。

### 98 说说 IdleHandler 

闲时机制，当消息队列空闲时会执行`IdelHandler`的`queueIdle()`方法，该方法返回一个`boolean`值，如果为`false`则执行完毕之后移除这条消息，如果为`true`则保留，等到下次空闲时会再次执行，

IdleHandler 是一个回调接口，可以通过MessageQueue的addIdleHandler添加实现类。

[IdleHandler 原理浅析](https://www.cnblogs.com/wytiger/p/13030199.html)

### 99 同步屏障机制(sync barrier)

Message 分为3种：

* 普通消息（同步消息）
* 屏障消息（同步屏障）
* 异步消息

我们通常使用的都是普通消息，而屏障消息就是在消息队列中插入一个屏障，在屏障之后的所有普通消息都会被挡着，不能被处理。不过异步消息却例外，屏障不会挡住异步消息，因此可以这样认为：**屏障消息就是为了确保异步消息的优先级，设置了屏障后，只能处理其后的异步消息，同步消息会被挡住，除非撤销屏障。**

同步屏障就是一个Message，一个target字段为空的Message。

同步屏障可以通过 MessageQueue.postSyncBarrier 函数来设置。该方法发送了一个没有 target 的 Message到Queue 中，在 next 方法中获取消息时，如果发现没有 target 的 Message，则在一定的时间内跳过同步消息，优先执行**异步消息**。再换句话说，同步屏障为 Handler 消息机制增加了一种简单的优先级机制，异步消息的优先级要高于同步消息。在创建 Handle r时有一个 async 参数，传 true 表示此 handler 发送的时异步消息。

ViewRootImpl.scheduleTraversals 方法就使用了同步屏障，保证UI绘制优先执行。


    void scheduleTraversals() {
    if (!mTraversalScheduled) {
    mTraversalScheduled = true;
    //设置同步障碍，确保mTraversalRunnable优先被执行
    mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
    //内部通过Handler发送了一个异步消息
    mChoreographer.postCallback(
    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
    if (!mUnbufferedInputDispatch) {
    scheduleConsumeBatchedInput();
    }
    notifyRendererOfFramePending();
    pokeDrawLockIfNeeded();
    }
    }




### 100 getWidth() 和 getMeasureWidth() 区别

* getMeasureWidth()：measure() 获取到，值通过setMeasuredDimension()方法来进行设置的
* getWidth()：layout() 获取到，值通过视图右边的坐标减去左边的坐标计算出来的。

### 101 requestLayout，invalidate，postInvalidate 之间的区别？

* requestLayout：会重新调用onMeasure、onLayout、onDraw 来刷新界面。
* invalidate：调用 onDraw() 来刷新界面。调用 invalidate()，会为该View添加一个标记位，同时不断向父容器请求刷新，父容器通过计算得出自身需要重绘的区域，直到传递到ViewRootImpl中，最终触发performTraversals方法，进行开始View树重绘流程(只绘制需要重绘的视图)。
* postInvalidate ：调用onDraw() 来刷新界面，在非UI线程中调用。

[Android View 深度分析requestLayout、invalidate与postInvalidate](https://blog.csdn.net/a553181867/article/details/51583060?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.baidujs&dist_request_id=1328592.24703.16148391637091151&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.baidujs)

### 102 apk安装流程

- 复制 APK 到 /data/app 目录下，解压并扫描安装包。
- 资源管理器解析 APK 里的资源文件。
- 解析 AndroidManifest.xml，并在 /data/data/ 目录下创建对应的应用数据目录。
- 对 dex 文件进行优化，并保存在 dalvik-cache目录下。
- 将 AndroidManifest.xml 解析出的四大组件信息注册到 PackageManagerService 中。
- 安装完成后，发送广播。

### 103 apk打包流程

![](../asset/apk签名过程.jpg)

* 打包资源文件，生成 R.java 文件
  - aapt 工具（aapt.exe） -> AndroidManifest.xml 和 布局文件 XMl 都会编译 -> R.java -> AndroidManifest.xml 会被 aapt 编译成二进制
  - res 目录下资源 -> 编译，变成二进制文件，生成 resource id -> 最后生成 resouce.arsc（文件索引表）
* 处理 aidl 文件，生成相应的 Java 文件
  * aidl 工具（aidl.exe）
* 编译项目源代码，生成 class 文件
* 转换所有 class 文件，生成 classes.dex 文件
  * dx.bat
* 打包生成 APK 文件
  * apkbuilder 工具打包到最终的 .apk 文件中
* 对APK文件进行签名
* 对签名后的 APK 文件进行对齐处理（正式包）
  * 对 APK 进行对齐处理，用到的工具是 zipalign

### 104 app 瘦身

* 代码混淆
* 使用 lint 去除无用资源
* 去除无用国际化支持
* 切一套图 xxhdpi 或 xhdpi
* 图片压缩， png 压缩或者使用webP图片
* 使用矢量图形，简单的图标可以使用矢量图片

### 105 64k

Android 应用 (APK) 文件包含 [Dalvik](https://source.android.google.cn/devices/tech/dalvik/) Executable (DEX) 文件形式的可执行字节码文件，这些文件包含用来运行应用的已编译代码。Dalvik Executable 规范将可在单个 DEX 文件内引用的方法总数限制为 65536，其中包括 Android 框架方法、库方法以及您自己的代码中的方法。

* **检查应用的直接依赖项和传递依赖项**
* **通过 R8 移除未使用的代码**

### 106 Android 系统架构

* 应用层
* 应用框架层
* 系统运行库层
* HAL
* Linux 内核层

### 107 遇到 Fragment 哪些问题？

* getActivity() 空指针：一般在异步任务中，Fragment 已经 onDetach()，可以使用全局变量 mActivity，onAttach() 方法里赋值。

* 视图重叠：重复加载了同一个 Fragment 导致重叠

     @Override 
     protected void onCreate(@Nullable Bundle savedInstanceState) {
     // 在页面重启时，Fragment会被保存恢复，而此时再加载Fragment会重复加载，导致重叠 ;
        if(saveInstanceState == null){
         // 或者 if(findFragmentByTag(mFragmentTag) == null)
           // 正常情况下去 加载根Fragment 
        } 
     }


  ### 108 View 绘制

  1. onMeasure：测量视图的大小，从顶层父View到子View递归调用measure()方法，measure()调用onMeasure()方法，onMeasure()方法完成测量工作。
  2. onLayout：确定视图的位置，从顶层父View到子View递归调用layout()方法，父View将上一步measure()方法得到的子View的布局大小和布局参数，将子View放在合适的位置上。
  3. onDraw：绘制最终的视图，首先ViewRoot创建一个Canvas对象，然后调用onDraw()方法进行绘制。onDraw()方法的绘制流程为：
     1. 绘制视图背景。
     2. 绘制画布的图层。
     3. 绘制View内容。
     4. 绘制子视图，如果有的话。
     5. 还原图层。
     6. 绘制滚动条。 

### 108 Binder 机制

传统的 Linux 进程通信。

1. 管道：在创建时分配一个page大小的内存，缓存区大小比较有限；
2. 消息队列：信息复制两次，额外的CPU消耗；不合适频繁或信息量大的通信；
3. 共享内存：无须复制，共享缓冲区直接付附加到进程虚拟地址空间，速度快；但进程间的同步问题操作系统无法实现，必须各进程利用同步工具解决；
4. 套接字：作为更通用的接口，传输效率低，主要用于不通机器或跨网络的通信；
5. 信号量：常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。
6. 信号: 不适用于信息交换，更适用于进程中断控制，比如非法内存访问，杀死某个进程等；

Android 的 Binder 机制

![](../asset/binder机制.jpg)

Binder通信的四个角色：

* Client进程：使用服务的进程。
* Server进程：提供服务的进程。
* ServiceManager进程：ServiceManager的作用是将字符形式的Binder名字转化成Client中对该Binder的引用，使得Client能够通过Binder名字获得对Server中Binder实体的引用。
* Binder驱动：驱动负责进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持。

#### 为什么Binder只进行了一次数据拷贝？

Linux 进程通信，需要先用 copy_from_user() 拷贝到内核空间，再用 copy_to_user()拷贝到另一个用户空间。

为了实现用户空间到用户空间的拷贝，mmap() 分配的内存除了映射进了接收方进程里，还映射进了内核空间。所以调用copy_from_user()将数据拷贝进内核空间也相当于拷贝进了接收方的用户空间，这就是Binder只需一次拷贝的‘秘密’。

最底层的是 Android的 ashmen(Anonymous shared memory) 机制，它负责辅助实现内存的分配，以及跨进程所需要的内存共享。AIDL(android interface definition language)对Binder的使用进行了封装，可以让开发者方便的进行方法的远程调用，后面会详细介绍。Intent是最高一层的抽象，方便开发者进行常用的跨进程调用。

从英文字面上意思看，Binder具有粘结剂的意思那么它是把什么东西粘接在一起呢？在Android系统的Binder机制中，由一系统组件组成，分别是Client、Server、Service Manager和Binder驱动，其中Client、Server、Service Manager运行在用户空间，Binder驱动程序运行内核空间。Binder就是一种把这四个组件粘合在一起的粘连剂了，其中，核心组件便是Binder驱动程序了，ServiceManager提供了辅助管理的功能，Client和Server正是Binder驱动和ServiceManager提供的基础设施上，进行Client-Server之间的通信。


参考阅读：

* [简单理解Binder机制的原理](https://blog.csdn.net/augfun/article/details/82343249)

* [Android跨进程通信：图文详解 Binder机制](https://blog.csdn.net/carson_ho/article/details/73560642)

* [写给 Android 应用工程师的 Binder 原理剖析](https://juejin.im/post/5acccf845188255c3201100f)

* [Binder学习指南](http://weishu.me/2016/01/12/binder-index-for-newer/)

* [Binder设计与实现](https://blog.csdn.net/universus/article/details/6211589)

* [老罗Binder机制分析系列或Android系统源代码情景分析Binder章节](https://blog.csdn.net/luoshengyang/article/details/6618363)

#### binder 驱动

* 从 Java 层来看就像访问本地接口一样，客户端基于 BinderProxy ，服务端基于 IBinder 对象
* 从 native 层来看来看客户端基于 BpBinder 到 ICPThreadState 到 binder 驱动，服务端由 binder 驱动唤醒 IPCThreadSate 到 BbBinder 。
* 跨进程通信的原理最终是要基于内核的，所以最会会涉及到 binder_open 、binder_mmap 和 binder_ioctl这三种系统调用。

### 109 如何解决View的事件冲突？

常见开发中事件冲突的有ScrollView与RecyclerView的滑动冲突、RecyclerView 内嵌同时滑动同一方向。

滑动冲突的处理规则：

- 对于由于外部滑动和内部滑动方向不一致导致的滑动冲突，可以根据滑动的方向判断谁来拦截事件。
- 对于由于外部滑动方向和内部滑动方向一致导致的滑动冲突，可以根据业务需求，规定何时让外部View拦截事件，何时由内部View拦截事件。
- 对于上面两种情况的嵌套，相对复杂，可同样根据需求在业务上找到突破点。

滑动冲突的实现方法：

- 外部拦截法：指点击事件都先经过父容器的拦截处理，如果父容器需要此事件就拦截，否则就不拦截。具体方法：需要重写父容器的 onInterceptTouchEvent 方法，在内部做出相应的拦截。
- 内部拦截法：指父容器不拦截任何事件，而将所有的事件都传递给子容器，如果子容器需要此事件就直接消耗，否则就交由父容器进行处理。具体方法：需要配合requestDisallowInterceptTouchEvent 方法。

### 110 SharePreference性能优化

SharePreferences是一个轻量级的存储类，特别适合用于保存软件配置参数。使用SharedPreferences保存数据，用 xml 文件存放数据，文件存放在/data/data/ < package name > /shared_prefs目录。

SharePreferences 在创建的时候会把整个文件全部加载进内存，如果SharedPreference文件比较大，会带来以下问题：

1. 第一次从sp中获取值的时候，有可能阻塞主线程，使界面卡顿、掉帧。
2. 解析sp的时候会产生大量的临时对象，导致频繁GC，引起界面卡顿。
3. 这些 key 和 value 会永远存在于内存之中，占用大量内存。

优化建议

1. 不要存放大的 key 和 value，会引起界面卡、频繁 GC、占用内存等等。
2. 毫不相关的配置项就不要放在在一起，文件越大读取越慢。
3. 读取频繁的 key 和不易变动的 key 尽量不要放在一起，影响速度，如果整个文件很小，那么忽略吧，为了这点性能添加维护成本得不偿失。
4. 不要乱 edit 和 apply，尽量批量修改一次提交，多次 apply 会阻塞主线程。
5. 尽量不要存放 JSON 和 HTML，这种场景请直接使用 JSON。
6. sp 无法进行跨进程通信，MODE_MULTI_PROCESS 只是保证了在 API 11 以前的系统上，如果 sp 已经被读取进内存，再次获取这个 sp 的时候，如果有这个 flag，会重新读一遍文件，仅此而已。

### 111 SQLite升级

数据库升级增加表和删除表都不涉及数据迁移，但是修改表涉及到对原有数据进行迁移。升级的方法如下所示：

1. 将现有表命名为临时表。
2. 创建新表。
3. 将临时表的数据导入新表。
4. 删除临时表。

重写

如果是跨版本数据库升级，可以由两种方式，如下所示：

1. 逐级升级，确定相邻版本与现在版本的差别，V1升级到V2,V2升级到V3，依次类推。
2. 跨级升级，确定每个版本与现在数据库的差别，为每个case编写专门升级大代码。

### 112 内存缓存和磁盘缓存是怎么实现的？

内存缓存基于 LruCache 实现，磁盘缓存基于 DiskLruCache 实现。这两个类都基于Lru算法和LinkedHashMap 来实现。


LRU 是Least Recently Used的缩写，最近最久未使用算法，它的核心原则是如果一个数据在最近一段时间没有使用到，那么它在将来被访问到的可能性也很小，则这类数据项会被优先淘汰掉。


为什么会选择LinkedHashMap 呢？

这跟 LinkedHashMap的特性有关，LinkedHashMap的构造函数里有个布尔参数 **accessOrder**，当它为 true 时，LinkedHashMap会以访问顺序为序排列元素，否则以插入顺序为序排序元素。

### 113 PathClassLoader与DexClassLoader有什么区别？

- PathClassLoader：只能加载已经安装到Android系统的APK文件，即/data/app目录，Android默认的类加载器。
- DexClassLoader：可以加载任意目录下的dex、jar、apk、zip文件。

### 114 WebView优化

出现原因：在客户端中，加载 H5 页面之前，需要先初始化WebView，在WebView完全初始化完成之前，后续的界面加载过程都是被阻塞的。

优化入手：

1. 预加载WebView。
2. 加载WebView的同时，请求H5页面数据。

常见方法：

1. 全局 WebView。
2. 客户端代理页面请求，WebView 初始化完成后向客户端请求数据。
3. asset 存放离线包。

除此之外还有一些其他的优化手段：

- 脚本执行慢，可以让脚本最后运行，不阻塞页面解析。
- DNS与链接慢，可以让客户端复用使用的域名与链接。
- React框架代码执行慢，可以将这部分代码拆分出来，提前进行解析。

### 115 什么是MeasureSpec？

MeasureSpec 代表一个32位int值，高两位代表SpecMode（测量模式），低30位代表SpecSize（具体大小）。

SpecMode有三类：

- AT_MOST：父容器指定一个可用大小即SpecSize，view 的大小不能大于这个值，具体多大要看view的具体实现，相当于wrap_content。
- EXACTLY： 父容器已经检测出view所需的精确大小，这时候view的最终大小SpecSize所指定的值，相当于 match_parent 或指定具体数值。
- UNSPECIFIED：表示父容器不对View有任何限制，一般用于系统内部，表示一种测量状态；

### 116 Fragment的懒加载实现

Fragment 可见状态改变时会被调用setUserVisibleHint()方法，可以通过复写该方法实现Fragment的懒加载，但需要注意该方法可能在onVIewCreated之前调用，需要确保界面已经初始化完成的情况下再去加载数据，避免空指针。



### 117 滚动歌词、高亮怎么实现？



### 118 动画是如何刷新父布局的？



### 119 如何画五子棋棋谱？



### 120 自定义 view(自定义view的时候，三个构造函数各自的作用)


    //在java代码创建视图的时候被调用，如果是从xml填充的视图，就不会调用这个
    public RoundProgressBar(Context context) {
            this(context, null);    
       }
 
    //在xml创建但是没有指定style的时候被调用
    public RoundProgressBar(Context context, AttributeSet attrs) {
           this(context, attrs, 0);　　
    }

     public RoundProgressBar(Context context, AttributeSet attrs, int defStyle) {}




### 121 Android对HashMap做了优化后推出的新的容器类是什么？

SparseArray
优点：它要比 HashMap 节省内存，某些情况下比HashMap性能更好，按照官方问答的解释，主要是因为SparseArray 不需要对 key 和 value 进行auto-boxing（将原始类型封装为对象类型，比如把int类型封装成Integer类型），结构比 HashMap 简单（SparseArray 内部主要使用两个一维数组来保存数据，一个用来存 key，一个用来存 value）不需要额外的额外的数据结构（主要是针对HashMap中的HashMapEntry而言的）。

### 122 WebView 漏洞

**任意代码执行漏洞**

* WebView 中 `addJavascriptInterface（）` 接口:当JS拿到Android这个对象后，就可以调用这个Android对象中所有的方法，包括系统类（java.lang.Runtime 类），从而进行任意代码执行。解决：4.2 版本之后：对被调用的函数以 `@JavascriptInterface`进行注解从而避免漏洞攻击。4.2 版本之前：采用**拦截prompt（）**进行漏洞修复。

* WebView 内置导出的 `searchBoxJavaBridge_`对象：在Android 3.0以下，Android系统会默认通过`searchBoxJavaBridge_`的Js接口给 WebView 添加一个JS映射对象：`searchBoxJavaBridge_`对象

  该接口可能被利用，实现远程任意代码。解决：删除`searchBoxJavaBridge_`接口。

* WebView 内置导出的 `accessibility` 和 `accessibilityTraversal`Object 对象：上同。

**密码明文存储漏洞**



    WebSettings.setSavePassword(false)  //关闭密码保存提醒


**域控制不严格漏洞**

当其他应用启动此 Activity 时， intent 中的 data 直接被当作 url 来加载（假定传进来的 url 为 file:///data/local/tmp/attack.html ），其他 APP 通过使用显式 ComponentName 或者其他类似方式就可以很轻松的启动该 WebViewActivity 并加载恶意url。


    //对于不需要使用 file 协议的应用，禁用 file 协议；
    // 禁用 file 协议；
    setAllowFileAccess(false); 
    setAllowFileAccessFromFileURLs(false);
    setAllowUniversalAccessFromFileURLs(false);
    
    //对于需要使用 file 协议的应用，禁止 file 协议加载 JavaScript。
    //需要使用 file 协议
    setAllowFileAccess(true); 
    setAllowFileAccessFromFileURLs(false);
    setAllowUniversalAccessFromFileURLs(false);

    // 禁止 file 协议加载 JavaScript
    if (url.startsWith("file://") {
        setJavaScriptEnabled(false);
    } else {
       setJavaScriptEnabled(true);
    }


### 123 Binder 同步与异步



### 124 WebSocket与socket的区别？



### 125 RecyclerView 缓存

一级缓存：屏幕内缓存（mAttachedScrap）
屏幕内缓存指在屏幕中显示的ViewHolder，这些ViewHolder会缓存在mAttachedScrap、mChangedScrap中 ：

mChangedScrap 表示数据已经改变的ViewHolder列表，需要重新绑定数据（调用onBindViewHolder）
mAttachedScrap 未与RecyclerView分离的ViewHolder列表

二级缓存：屏幕外缓存（mCachedViews）
用来缓存移除屏幕之外的 ViewHolder，默认情况下缓存容量是 2，可以通过 setViewCacheSize 方法来改变缓存的容量大小。如果 mCachedViews 的容量已满，则会优先移除旧 ViewHolder，把旧ViewHolder移入到缓存池RecycledViewPool 中。

三级缓存：自定义缓存（ViewCacheExtension）
给用户的自定义扩展缓存，需要用户自己管理 View 的创建和缓存，可通过Recyclerview.setViewCacheExtension()设置。

四级缓存：缓存池（RecycledViewPool ）
ViewHolder 缓存池，在mCachedViews中如果缓存已满的时候（默认最大值为2个），先把mCachedViews中旧的ViewHolder 存入到RecyclerViewPool。如果RecyclerViewPool缓存池已满，就不会再缓存。从缓存池中取出的ViewHolder ，需要重新调用bindViewHolder绑定数据。

按照 ViewType 来查找 ViewHolder
每个 ViewType 默认最多缓存 5 个
可以多个 RecyclerView 共享 RecycledViewPool
RecyclerViewPool底层是使用了SparseArray来分开存储不同ViewType的ViewHolder集合

### 126 版本适配

6.0 ：动态权限

7.0：FileProvider

8.0：通知权限、安装 APK

9.0：前台服务添加权限

10.0：后台运行时访问设备位置信息需要权限

11.0：用户对位置信息的控制，方法是添加了一次性权限

[Android版本差异适配方案(5.0-11.0)](https://blog.csdn.net/yinhaide/article/details/103295050)



### 127  Binder特点



### 128 说说 AIDL 流程（字节面试）



### 129 说说 Kotlin 中的内联函数？



## drawable设置阴影 ##

    <?xml version="1.0" encoding="utf-8"?>
    <layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape android:shape="rectangle">
            <corners android:radius="@dimen/y30" />
            <gradient
                android:angle="270"
                android:endColor="@color/color_3f8473"
                android:startColor="@color/colorPrimary"
                android:centerX="0.5"
                android:centerY="0.5"
                android:type="linear" />
        </shape>
    </item>
    <item
        android:bottom="2dp"
        android:left="4dp"
        android:top="@dimen/x9">
        <shape android:shape="rectangle">
            <solid android:color="@color/Red" />
            <corners android:radius="@dimen/y30" />
        </shape>
    </item>

    </layer-list>


## 设置文本阴影 ##
  
**字体阴影需要四个相关参数：**

1. android:shadowColor：阴影的颜色
2. android:shadowDx：水平方向上的偏移量
3. android:shadowDy：垂直方向上的偏移量
4. android:shadowRadius：是阴影的的半径大少 0 是没有，值越大阴影越大越模糊

**dx  dy 分别为   （0 ， 0） ， （10 ， 10 ） ， （30 ， 30）**

结果如下：
![](https://img-blog.csdn.net/20141011142658278?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGV3ZW5jZTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**dx  dy 都是 30  shadowRadius  分别为： 0 ， 0.01 ， 1 ， 2 ， 5 ， 10**

结果：
![](https://img-blog.csdn.net/20141011143555964?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGV3ZW5jZTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

调试dx  跟 dy来改变光源，使阴影偏向不同的方向 跟  距离

 如果光源是在左边，那么dx 是为正的， 

光源在最右边，那么dx就是负

光源在上  那么dy 为 正

光源在下， 那么dy 为 负  

**shadowRadius 都是 1  （5,0） ; （-5,0） ;（0,5） ;（0，-5） ;**

如下：

![](https://img-blog.csdn.net/20141011145513484?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGV3ZW5jZTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

带一点浮雕效果：把dx  dy都设置较小的值
设置为 （0.2 ， 0.2） ， （1 ， 1） ， （2 ， 2）

![](https://img-blog.csdn.net/20141011150421656?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGV3ZW5jZTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

荧光灯的效果： 把把dx  dy设置为0 ， Raduis位置较大就行了，最重要的事字体颜色 跟背景颜色一样（或者非常相近）

## 创建Dialog对象依赖的Context必须是Activity吗？

对于这个问题，回答肯定是：不是的。

在创建Dialog对象时，context参数传Activity和传Service或Application之类的非Activity的Context对象有什么区别呢？

有经验的同学会说，想要通过非Activity对象创建并正常显示Dialog，首先必须拥有SYSTEM_ALERT_WINDOW权限，还有，在调用Dialog.show方法之前，必须把Dialog的Window的type指定为SYSTEM_WINDOW类型，比如TYPE_SYSTEM_ALERT或TYPE_APPLICATION_OVERLAY。

没有满足第一个条件的话，那肯定会报permission denied啦。

如果在show之前没有指定Window的type为SYSTEM_WINDOW类型，一样会发生BadTokenException的，message是token null is not valid; is your activity running?。


##LayoutInflater.Factory2

## Activity的启动流程

1. Activity启动流程。其中考察最多的类似问题是：【从桌面点击一个图标之后，到界面显示，这个过程发生了什么？】。很多时候面试官会结合activity生命周期来考问：在启动流程的哪些阶段哪些生命周期被回调，此时Activity状态如何。
1. 启动模式。也就是常见的四种启动模式，但面试官更喜欢问何时使用他们，也就是使用场景。
1. 生命周期。这个很少单独问，一般和启动流程或者具体的业务场景结合考问。
1. context。主要是内存泄露的考察以及application和activity两种context如何选择。
## 事件分发
[https://blog.csdn.net/guolin_blog/article/details/9097463](https://blog.csdn.net/guolin_blog/article/details/9097463 "事件分发")
1. dispatchTouchEvent ：处在链首，用于分发事件，该方法决定是由当前View自己的onTouchEvent来处理，还是分发给子View，让子View递归调用其自身的dispatchTouchEvent来处理。
1. onInterceptTouchEvent ：是用来拦截事件的，当父控件下发事件给子控件进行拦截处理的时候，如果子控件需要对事件进行处理，就要在onInterceptTouchEvent方法中进行拦截，然后到子控件的onTouchEvent方法中进行事件的监听以及逻辑的判断。
1. onTouchEvent ：用于处理传递到View的手势事件。

当我们点击屏幕的时候，就会产生Event事件，此时会首先调用Activity的dispatchTouchEvent方法，源码如下：

        public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }

从上面的代码我们可以看出，如果是ACTION_DOWN事件，会调用onUserInteraction()方法，onUserInteraction方法的作用是当触屏点击按home，back，menu键等都会触发此方法。下拉statubar、旋转屏幕、锁屏不会触发此方法，所以可以用在屏保应用上。


Activty事件分发很简单，如果重写了dispatchTouchEvent方法，无论是返回值是false还是true，此时事件分发结束，即不再向下传递，只有返回super.dispatchTouchEvent()时，才会正常往下传递；

     /**
     * Pass the touch screen motion event down to the target view, or this
     * view if it is the target.
     *
     * 将屏幕的按压事件传递给目标view，或者当前view即目标view
     * @param event The motion event to be dispatched.
     * @return True if the event was handled by the view, false otherwise.
     */

    public boolean dispatchTouchEvent(MotionEvent event) {
    //最前面这一段就是判断当前事件是否能获得焦点，如果不能获得焦点或者不存在一个View，那我们就直接返回False跳出循环
        // If the event should be handled by accessibility focus first.
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.
            event.setTargetAccessibilityFocus(false);
        }

    //设置返回的默认值
        boolean result = false;

    //这段是系统调试方面，可以直接忽略
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

    //Android用一个32位的整型值表示一次TouchEvent事件,低8位表示touch事件的具体动作，比如按下，抬起，滑动，还有多点触控时的按下，抬起，这个和单点是区分开的，下面看具体的方法: 
    //1 getAction:触摸动作的原始32位信息，包括事件的动作，触控点信息 
    //2 getActionMasked:触摸的动作,按下，抬起，滑动，多点按下，多点抬起 
    //3 getActionIndex:触控点信息
        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
     // 当我们手指按到View上时,其他的依赖滑动都要先停下
            stopNestedScroll();
        }

    //过滤掉一些不合法的事件,比如当前的View的窗口被遮挡了。  
        if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }

    //ListenerInfo 是view的一个内部类 里面有各种各样的listener,例如OnClickListener，OnLongClickListener，OnTouchListener等等

            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
    //首先判断如果监听li对象!=null 且我们通过setOnTouchListener设置了监听，即是否有实现OnTouchListener，如果有实现就判断当前的view状态是不是ENABLED,如果实现的OnTouchListener的onTouch中返回true，并处理事件，则

            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
    //如果满足这些条件那么返回true，这个事件就在此处理
    // 意味着这个View需要事件分发 
                result = true; 
            }

    //如果上一段判断的条件没有满足（没有在代码里面setOnTouchListener的话），就判断View自身的onTouchEvent方法有没有处理，没有处理最后返回false，处理了返回true；
            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

    //系统调试分析相关，没有影响
        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
    ////如果这是手势的结尾，则在嵌套滚动后清理
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }
    
如果`dispatchTouchEvent`返回true，说明以下代码执行

     if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }
此时`onTouch`返回值为true，说明走了onTouchEvent方法


当触摸事件发生时，首先 Activity 将 TouchEvent 传递给最顶层的 View，TouchEvent最先到达最顶层 view 的 dispatchTouchEvent ，然后由 dispatchTouchEvent 方法进行分发，

如果dispatchTouchEvent返回true 消费事件，事件终结。

如果dispatchTouchEvent返回 false ，则回传给父View的onTouchEvent事件处理；

如果dispatchTouchEvent返回super的话，默认会调用自己的onInterceptTouchEvent方法。

- 默认的情况下onInterceptTouchEvent回调用super方法，super方法默认返回false，所以会交给子View的onDispatchTouchEvent方法处理

- 如果 interceptTouchEvent 返回 true ，也就是拦截掉了，则交给它的 onTouchEvent 来处理，
- 如果 interceptTouchEvent 返回 false ，那么就传递给子 view ，由子 view 的 dispatchTouchEvent 再来开始这个事件的分发。

###滑动冲突
####外部解决法
从父View着手，重写onInterceptTouchEvent方法，在父View需要拦截的时候拦截，不要的时候返回false，为代码大概 如下

     @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
    final float x = ev.getX();
    final float y = ev.getY();

    final int action = ev.getAction();
    switch (action) {
        case MotionEvent.ACTION_DOWN:
            mDownPosX = x;
            mDownPosY = y;

            break;
        case MotionEvent.ACTION_MOVE:
            final float deltaX = Math.abs(x - mDownPosX);
            final float deltaY = Math.abs(y - mDownPosY);
            // 这里是够拦截的判断依据是左右滑动，读者可根据自己的逻辑进行是否拦截
            if (deltaX > deltaY) {
                return false;
            }
    }

    return super.onInterceptTouchEvent(ev);
    }




####内部解决法
从子View着手，父View先不要拦截任何事件，所有的事件传递给 子View，如果子View需要此事件就消费掉，不需要此事件的话就交给 父View处理。
实现思路 如下，重写子 View的dispatchTouchEvent方法，在Action_down 动作中通过方法 requestDisallowInterceptTouchEvent（true） 先请求 父 View不要拦截事件，这样保证子 View 能够接受到 Action_move 事件，再在 Action_move 动作中根据自己的逻辑是否要拦截事件，不需要拦截事件的话再交给 父 View 处理。

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
    int x = (int) ev.getRawX();
    int y = (int) ev.getRawY();
    int dealtX = 0;
    int dealtY = 0;

    switch (ev.getAction()) {
        case MotionEvent.ACTION_DOWN:
            dealtX = 0;
            dealtY = 0;
            // 保证子View能够接收到Action_move事件
            getParent().requestDisallowInterceptTouchEvent(true);
            break;
        case MotionEvent.ACTION_MOVE:
            dealtX += Math.abs(x - lastX);
            dealtY += Math.abs(y - lastY);
            Log.i(TAG, "dealtX:=" + dealtX);
            Log.i(TAG, "dealtY:=" + dealtY);
            // 这里是够拦截的判断依据是左右滑动，读者可根据自己的逻辑进行是否拦截
            if (dealtX >= dealtY) {
                getParent().requestDisallowInterceptTouchEvent(true);
            } else {
                getParent().requestDisallowInterceptTouchEvent(false);
            }
            lastX = x;
            lastY = y;
            break;
        case MotionEvent.ACTION_CANCEL:
            break;
        case MotionEvent.ACTION_UP:
            break;

    }
    return super.dispatchTouchEvent(ev);
}

##Handler
1. 内部原理。Handler必会的啊，android的消息机制，可以称为android程序的引擎来的。
1. 同步屏障。涉及到绘制优化、屏幕刷新机制等。
1. 阻塞唤醒原理。这里一般是会问为何loop()方法是死循环却不会占用cpu时间片 or 为何next()方法阻塞却不会卡死。更深一点会问到Linux的IO多路复用epoll原理。
1. 卡顿与内存优化。整个主线程的所有任务都必须经过Looper，是排查卡顿和ANR的关键点，以及消息太多会造成的后果等。
1. 消息复用。


##图片
1. 计算一张图片大小。分辨率x像素点大小，考察得很多次。
1. 加载优化。如LaunchActivity同时加载的图片太多如何优化、view的大小比图片小如何优化等。
1. 缓存优化。内存缓存、硬盘缓存。
1. Glide框架。重点就是Glide对上面的优化的实现原理，因此需要着重看Glide的缓存原理。
1. drawable。对比使用图片和drawable的好处，以及drawable的原理。
##SharePreference
1. 内部原理
1. commit和apply的区别
1. 这部分考察得不多，但建议读者可以深入理解sp的缺点，如导致ANR原理，以及新框架MMKV、Data Store的优点。

##window
主要window的类型、以及window的真正定义理解即可。

##view
1. 事件分发流程。考察得最多，基本把整个分发流程讲清楚就好了；其次还会考察如何解决具体的冲突场景。
1. 应用界面的view层级。
1. 绘制流程与时机。activity启动时到onResume方法被调用，view依旧还未被绘制。

##IPC
1. 常见IPC类型以及优缺点。
1. Binder机制。优点、缺点、特点、和传统IPC比较。Binder涉及到的很多是偏底层，更多的时候考察的是上层的应用，如和socket比较等。
1. socket。这个会重点问，涉及优缺点、使用场景、和binder相比等。

##序列化
serializable和parcelable的原理，以及各自的优缺点、应用场景。

##jetpack
使用jetpack的好处与坏处。
框架原理。这部分如果写在简历也很少问，看面试官；但如果问的话，一般会问原理。

##okHttp

1. 拦截器以及责任链思想。这个是问的最多的。
1. 内部调度器对线程、任务数的并发控制。
1. 优点缺点，和URLConnection相比的好处，诞生的背景等。

##优化
1. 性能优化。问得很多，主要看自己平时有没有做过优化。
1. 卡顿优化。一般询问如何定位和解决卡顿问题。
1. 内存优化。一般是内存泄露、或者减少内存占用等。
1. ANR。一般会考察原理以及如何解决。
1. 工具：leakcanary、profiler。优化涉及的内容太多，这个属于比较深的内容，还是得看自己平时的项目积累。


#Kotlin

##协程
问的是最多的，主要是理解线程和协程的关系、协程的优缺点。这个也看个人的学习程度了。

##run、let、also、with、apply

注意返回值以及作用域

##特性的具体实现
如默认参数的具体实现。有了解过最好了，没有的话就联想Java是如何实现的，如默认参数可以联想方法重载。

#计算机网络

计网这一部分主要还是HTTP和TCP的内容了，经典中的经典。需要注意的是链路层、网络层的一些协议要了解，也是计算机基础了，被问到不会的话会比较尴尬。其次是一些新的协议如QUIC、http3.0等可以了解一下，面试会很加分，同时也可以进一步去理解TCP的优缺点。


##HTTP
http在android上的体现并不多，因为大部分的工作都给框架解决了，问的问题其实很少。

1. 历代http的优化以及原因
1. 请求方法、响应码

##HTTPS

必问。一定要会了

1. 原理以及和http的区别。加密算法、hash摘要、ca证书验证都要了解
1. 建立连接过程
1. 破解：中间人攻击等

##数据链路层、IP层

1. ARP和RARP协议
1. NAT协议
1. DNS
1. 这部分主要问一些常见的协议，考察计网功底，这里列出来的是笔者考察过的，读者需要比价系统地去学习这一块。

##TCP

1. 握手挥手
1. 拥塞控制
1. 可靠传输原理
1. 缺点以及如何改进。这个是比较重要的，对应http3.0的优化就是针对TCP的缺点来入手的。
1. TCP的连接数目上限
1. TCP非常重要，必问的内容，不会的读者一定要去好好学习一下。

##UDP

1. 优缺点
1. 和TCP比较
1. 应用场景
1. 一般和TCP一起出现，询问他们的区别，以及如何通过UDP来优化TCP的缺点。

##数据格式

json的优缺点，为什么要使用json而不是XML。熟记就可以了。