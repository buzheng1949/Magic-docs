#### 一、什么是ThreadLocal
- **ThreadLocal**

在网上许多的文章都是这么解释的，ThreadLocal提供了一种多线程并发处理共享变量的一种安全的解决思路。但是仔细查看代码之后，发现ThreadLocal更准确来讲，是提供了一种不同的线程之间，彼此的变量相互隔离，互不干扰的一种便于线程自己变量使用的方式。

- **简单例子**

假如有一个银行，银行对客户的钱是存放在每个客户的存款账号里面，不同的客户对自己的钱进行消费存储。

okay，我们类比一下ThreadLocal，当银行是一个ThreadLocal，每个用户是一个线程，ThreadLocal往不同的客户（线程）创建不同的账户（变量副本），然后每个用户存取钱的时候，实际上是往自己的账户存取钱，那么也就不会影响到其他用户的账户了。

我们再拓展一下，上面我们说了一个银行，但是银行很多呀，那也就意味着每个线程都是可以有多个ThreadLocal的，所以理解下来就变成了，线程里面存着多个由不同的ThreadLocal存储着对应的变量副本。
关系如下图所示：
![ThreadLocal关系图](http://upload-images.jianshu.io/upload_images/1428117-8346ff36fbe906e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 二、ThreadLocal有什么作用？
举个例子，分析一下ThreadLocal的作用。

假如我们有这样的一个场景，我们需要对用户的操作日志进行存储，用户的请求可能同一时刻打过来的，假设我们只有一台服务器，这台服务器处理该请求的tps是100，每个请求处理时间是10ms，一个线程可以处理10个tps，那我们可以知道服务器同时开启的线程数应该是10个。那并发情况下，10个请求同时打过来，不同请求用户的访问参数是不一样的，比如用户的ID，昵称，业务类型都是不一样的，所以对于我们需要记录用户操作而言，每个线程都需要一个自己的变量副本才不会跟其他线程因为修改了同样的变量而导致不安全的现象，那么ThreadLocal就可以派上用场了，因为ThreadLocal会在这十个线程里面设置变量副本，不同线程里面的变量副本自己维护，所以就不会存在数据竞争的情况发生。

- 通过一个小例子验证，ThreadLocal确实是能隔离变量的

```
 package com.buzheng.threadlocaltest;

/**
 * created by buzheng
 * ThreadLocal测试demo
 */
public class ThreadLocalTest {

    private ThreadLocal<Long> longThreadLocal = new ThreadLocal<>();

    private ThreadLocal<String> stringThreadLocal = new ThreadLocal<>();

    public void setThreadLocalVaule() {
        longThreadLocal.set(Thread.currentThread().getId());
        stringThreadLocal.set(Thread.currentThread().getName());
    }

    public Long getThreadId() {
        return longThreadLocal.get();
    }

    public String getThreadName() {
        return stringThreadLocal.get();
    }

    public static void main(String[] args) {
        ThreadLocalTest threadLocalTest = new ThreadLocalTest();
        threadLocalTest.setThreadLocalVaule();
        show("the threadLocal's name is " + threadLocalTest.getThreadId() + "   the threadLocal's id is " + threadLocalTest.getThreadName());
        new Thread(new Runnable() {
            @Override
            public void run() {
                threadLocalTest.setThreadLocalVaule();
                show("the threadLocal's name is " + threadLocalTest.getThreadId() + "   the threadLocal's id is " + threadLocalTest.getThreadName());
            }
        }).start();
    }

    public static void show(Object object) {
        System.out.println(object);
    }
}
```

 上述代码的输出结果是![运行结果](http://upload-images.jianshu.io/upload_images/1428117-80f21cdaa413f53a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 我们可以看到，确实不同线程里面的变量是独立维护的，主线程跟子线程里自己有自己的一份变量副本，自己维护，互不打扰。
 
####  三、ThreadLocal到底是如何工作的
在ThreadLocal的源码中，我们可以看到，ThreadLocal的类定义是酱紫的

```
/**
 * This class provides thread-local variables.  These variables differ from
 * their normal counterparts in that each thread that accesses one (via its
 * {@code get} or {@code set} method) has its own, independently initialized
 * copy of the variable.  {@code ThreadLocal} instances are typically private
 * static fields in classes that wish to associate state with a thread ...
 */
```
简单翻译一下：ThreadLocal提供了线程自己的本地变量。当不同线程访问这些变量的时候，每个线程访问的变量都是不一样的，访问的变量是取决于线程的变量副本。ThreadLocal实例通常来说都是private static类型的，用于关联线程和线程的上下文。

那简单看一下ThreadLocal是如何工作的，如果我们自己设计的话，最简单的做法就是我们给每个ThreadLocal类创建一个Map<Long,Object>，key是Thread的ID，Object是需要存储的线程的数据，但是Java的设计并非这么简单，而且刚好是相反的。

 - get（）方法
 
   ```
    /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
   ```
   从上面我们可以看到get()方法做了以下的事情
   - 获取当前的线程t
   - 调用getMap(t)方法获取当前线程的所有的变量副本ThreadLocalMap
   - 从ThreadLocalMap取出当前ThreadLocal的数据返回
   - 如果值为空的话，说明未设置，那么调用setInitialValue进行初始化。
   
 - 看一下ThreadLcolMap是什么东西
 
   ```
    /**
     * Get the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param  t the current thread
     * @return the map
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
   ```
   在代码里面我们可以看到，getMap返回的是ThreadLocal关联的Map，简单来说ThreadLocalMap就是跟当前线程存储的一个映射表，这个映射表存储了当前线程所有的ThreadLocal的数据。
   我们再往下看一下ThreadLocalMap类的定义，我们可以看到ThreadLocalMap是ThreadLocal的一个内部类。这个内部类的构造函数如下：
   
   ```
      /**
         * Construct a new map initially containing (firstKey, firstValue).
         * ThreadLocalMaps are constructed lazily, so we only create
         * one when we have at least one entry to put in it.
         */
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }

   ```
   我们惊奇的发现，ThreadLocalMap就是用ThreadLocal作为key的去设计的。
   那么我们就可以串联起来，ThreadLocal的实现的大致流程是：
   - 在一个线程里面维护一个ThreadLocalMap
   - ThreadLocalMap里面可以有多个ThreadLocal，每个ThradLocal作为Key进行存储，将ThreadLocal的变量副本作为value进行保存。
   
  **所以刚才说，我们最简单的实现方式是：每个ThreadLocal用一个Map<ThreadID,Vaule>的存储方式，Java的实现刚好相反，是通过线程里持有ThreadLocalMap<ThreadLocal,Vaule>的方式，然后通过Thread找到对应的ThreadLocalMap的方式去实现的**
  
#### 四、写在最后的总结
 - 每个线程是可以拥有多个ThreadLocal的，每个ThreadLocal跟ThreadLocal的值存在线程的ThreadLocalMap里面（这也是为什么要用ThreadLocal作为Key的原因）
 - 在调用get方法的时候，需要先set，不然会报空指针异常，不然需要自己重写ThreadLocal的initialValue()方法，如下：
 
 ```
 ThreadLocal<Object> threadLocal = new ThreadLocal<Object>(){
        @Override
        protected Object initialValue() {
            return new Object();
        }
    };
 ```
 
 - ThreadLocal存在的意义不是说为了解决多线程情况下，共享变量的同步问题，而更准确的说是为了解决当变量之间不相互依赖，在一个线程情况下不同的方法进行调用的问题。好比如我们的银行卡账户在不同的ATM机都可以取钱这样的case。我们有一个ThredLocalMap放着我们多个银行账户的副本，然后我们取什么银行的钱，往什么银行存钱都不会互相影响了，逃~~~ 
 

  

 

