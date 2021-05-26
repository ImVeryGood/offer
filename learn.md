# 一：Java基础 #

## 装箱拆箱
装箱就是  自动将基本数据类型转换为包装器类型；拆箱就是  自动将包装器类型转换为基本数据类型

**面试：**

下面这段代码的输出结果是什么？
    `public class Main {
    public static void main(String[] args) {
         
        Integer i1 = 100;
        Integer i2 = 100;
        Integer i3 = 200;
        Integer i4 = 200;
         
        System.out.println(i1==i2);输出true
        System.out.println(i3==i4); 输出false
    }
    }`

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

1、使用ReentrantLock实现同步

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