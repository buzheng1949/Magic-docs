##### 一、概念

  Java内存模型（Java Memory Model）的可见性描述的是Java程序中线程共享的变量的访问规则，以及在JVM中将变量存储到内存以及从内存中读取变量的底层细节。
 
##### 二、Java共享变量的可见性

  Java中的共享变量需要被**其他线程**访问的时候，该变量会拷贝一份**变量副本**到访问该线程的**工作内存**中，如下图所示：
  ![Java多线程变量访问规则](http://upload-images.jianshu.io/upload_images/1428117-c8e6162921dcc842.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  从上图我们可以看到以下两点:
  
  1、线程对于共享变量的操作规定只能在该线程的工作内存中进行，而不能直接从主内存中进行读写操作.
 
  2、 不同线程之间彼此的工作内存中的变量是不能直接被访问的，每个线程对自己的工作内存负责，而线程之间变量值的传递需要通过**主内存**这个中心枢纽来完成。

##### 三、共享变量可见性的原理实现
 
 假如我们有这样的case，变量x在线程1进行了修改，而线程2也需要进行及时看到变量x的变化情况，那么需要以下的步骤：
   1、线程1中的工作内存中更新的共享变量刷新到主内存中。
 2、 将主内存中最新的共享变量刷新到线程2的工作内存中。
 
- 如何保证共享变量的可见性
  - 线程修改后的共享变量能及时从自己的工作内存中刷新到主内存中。
  - 其他线程能及时的将共享变量的最新值从主内存中更新到自己的工作内存中。

- 可见性的实现方式
   - synchronized实现可见性
   
   我们一直知道synchronized是加锁的操作，但是实际上，JMM对synchronized的规定是：线程解锁之前，必须把共享变量的最新值刷新到主内存中，并把线程的变量标注为失效。而线程加锁的时候，将工作内存中的标记为失效的共享变量的值从主内存重新读取最新的值。
  
   -  volatile
      -  可见性
volatile实现内存的可见性是通过加入内存屏障以及进制重排序的优化进行实现的。对volatile变量进行写操作的时候，会在写操作后面加入一条**store**指令，该指令强制变量写操作之后一定要从工作内存中刷新到主内存中。而当volatile变量进行读操作的时候，会在读操作的前面加入**load**指令，该指令强制变量读操作之前一定要从主内存中更新变量的值。
        - 原子性：volatile不能保证volatile变量复合操作的原子性，举个简单的例子，如下面的例子:
   
   ```
   private volatile int age = 10;
   age ++;
   ```
   其中，age++是需要经过三个过程的，第一个是从内存中读取age的值，第二是将age的值加1，第三是将age最新的值写入内存中。在这三个过程中，多线程情况下依旧可能会在执行第一步或者第二步的时候让出CPU的执行权，而导致变量被其他的线程所操作，发生不安全的问题。所以如果是需要保证原子性的时候，需要加入synchronized关键字。
   简单代码如下:

```
 /**
 * Created by buzheng on 17/12/3.
 */
public class TestVolatile {

    private volatile int age = 0;

    public int getAge() {
        return age;
    }

    public void increaseAge() {
        age++;
    }

    public static void main(String[] args) {
        final TestVolatile testVolatile = new TestVolatile();
        for (int i = 0; i < 100; i++){
            new Thread(new Runnable() {
                @Override
                public void run() {
                    testVolatile.increaseAge();
                }
            }).start();
        }

        //如果超过2个线程在运行，让出CPU资源直到所有的子线程执行完
        while (Thread.activeCount() > 2){
            Thread.yield();
        }

        System.out.print("the result is "+ testVolatile.getAge());
    }
}
   ```
   
   我们可以看到，如果运行这段代码是有可能出现result<1000的，原因就是上面说的volatile对于对于变量的非原子操作同样是不安全的。解决volatile变量非原子操作不安全的问题，主要有以下方式:
   
   第一：加synchronized
   
   因为前面说了，synchronized关键字是可以保证可见性以及原子性的，既然volatile是可以保证可见性的，那么我们只需要通过synchronized保证操作的原子性即可。
   
   第二：lock
   lock也是跟synchronized异曲同工，也是加锁，注意应该释放锁的操作。代码如下：
   
```
/**
 * Created by buzheng on 17/12/3.
 */
public class TestVolatile {

    private volatile int age = 0;

    Lock lock = new ReentrantLock();

    public int getAge() {
        return age;
    }

    public void increaseAge() {
        //第一个方式，加锁，可以保证age变量操作的原子性
//        synchronized (this){
//            age++;
//        }
        //第二个方式
        try {
            lock.lock();
            age++;
        } finally {
            lock.unlock();//释放锁
        }


    }

    public static void main(String[] args) {
        final TestVolatile testVolatile = new TestVolatile();
        for (int i = 0; i < 100; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    testVolatile.increaseAge();
                }
            }).start();
        }

        //如果超过1个线程在运行，让出CPU资源直到所有的子线程执行完
        while (Thread.activeCount() > 2) {
            Thread.yield();
        }

        System.out.print("the result is " + testVolatile.getAge());
    }


}
```
   
   
   
  
##### 四、synchronized与volatile比较

- volatile不需要加锁 ，更加轻量级，不会阻塞线程。但只能保证可见性，不能保证原子性。
- synchronized是通过加锁的方式进行保证可见性以及原子性的，所以会阻塞线程。但是使用场景更加广泛。  
   


