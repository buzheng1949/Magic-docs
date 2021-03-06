#### 一、简述
  Java的内存管理实际上解决了两个问题
  
  - 内存分配
  - 内存回收
  
  接下来分两部分学习Java的内存分配以及回收策略
  
#### 二、Java垃圾收集

##### 1、垃圾收集需要解决的问题
  
   在使用Java的时候，Java为我们提供了垃圾回收，使我们不需要去手动去进行对象的释放。但是垃圾回收是一门大学问，主要有以下三个问题：
   
   - 回收什么
   - 怎么回收
   - 几时回收
   
##### 2、对象是否已经可以回收
    
   在进行垃圾回收的时候，首先需要考虑的是我们的对象哪些可以进行回收，哪些不能进行回收。
   
   - 引用计数
   
   在大多数的情况下，引用计数是有效果的，它也非常的简单，当对象上面有引用的时候，每增加一个引用，就增加对象上存在引用的个数，当对象减少引用的时候，就进行减一的操作，当计数为0的时候，说明该对象没有被其他对象引用到，可以进行回收。但是该方法存在的问题是，无法解决循环引用的问题。以下面的代码为例子：
   
 ```
 /**
 * Created by buzheng on 17/12/6.
 * 测试循环引用
 */
public class MemoryTest {

    public Object instance = null;

    public static void testGCRoot() {
        MemoryTest memoryTest1 = new MemoryTest();
        MemoryTest memoryTest2 = new MemoryTest();
        memoryTest1.instance = memoryTest2;
        memoryTest2.instance = memoryTest1;
        memoryTest1 = null;
        memoryTest2 = null;
       
    }
}
 ```
  此时两个对象彼此之间相互持有，但是实际上它们已经没用了，引用计数法无法解决该类问题。
  
  - 可达性分析
  
  可达性分析解决了引用计数的弊端，它的思路是用GCRoot作为根节点，如果某个对象是有用的，那么总能找到属于它跟GCRoot的链路，如果某个对象，无法找到抵达GCRoot的路径，那么说明该对象是不可达的，可以进行回收操作。
  
  如下图所示:![可达性分析（图来自深入理解Java虚拟机）](https://s10.mogucdn.com//mlcdn/c45406/171206_8bc1bc9397f99c9i86914hag0i9f3_1528x942.png)
  
  从上图可以看到，object5等是没有链路到达GCRoot的，说明对象是不可达的，即使它存在引用，也是可以进行垃圾回收的。
  
##### 3、使用什么算法进行回收

 - 标记清除算法
 
 标记清除算法是一种简单直接的清除算法，假设内存分为N块，在N块中如果分散着M块的区域是可以进行回收的，那么对着M块进行标记后回收。存在的问题主要是，因为内存是分散的，如果仅仅进行简单的标记清除的话，会造成内存碎片，也就是无法形成一整片的内存空间，假设有一个对象需要分配，而无法找到足够大的内存空间进行分配的时候，又会触发GC操作，所以JVM采用的都是基于标记清除算法改进的算法。
 
- 标记整理算法

  标记整理算法指的是当在内存中，将存活的对象进行标记并整理到一块指定的内存中，然后对需要回收的对象进行回收。
  
- 复制算法

 每次只用一半的内存进行使用，当需要回收的时候，将存活的对象移动到另外一半未使用的内存中，然后将需要回收的内存进行回收。
  
  明显的是不同的算法适用于Java的不同对象分配的区域，在Java中，对象分配到两个区域，一个是新生代，一个是老年代，因为新生代是来得快去得快，所以比较适合复制算法，当然复制回收不是按照1：1的比例进行设置。而老年代是经过多次GC还存活下来的对象，说明是比较稳定的，可以采用标记整理算法。
  
#### 三、对象的内存分配
  前面已经讲了对象何时进行回收以及使用什么方式进行回收，那么接下来讲的是对象如何进行内存的分配，我们的对象总得有个可以住的地方才行。
  
  Java对象分配的分配主要如下图：
  ![Java内存分配](https://s10.mogucdn.com//mlcdn/c45406/171206_00b0j866ekie6l7d50987f0529adl_1014x456.png)
  
  
  从上图可以看到，内存分配主要分为新生代跟老年代。
  
  - 新生代
  
  	- Eden区
  	- Survior区（又分为From跟To两个区域）
  
  - 老年代（经过多次GC的对象以及数组）
  
对象分配的原则主要有以下几个：

- 对象优先在Eden区域进行分配：当有对象需要分配的时候，大多数情况下会在Eden区进行分配，当Eden区域不够空间分配的时候，会触发MinorGC，假如做这样的假设：我们设置Xmx，Xms跟Xmn分别是20，20 ，10 ，说明我们的堆大小刚好是20，然后新生代大小为10，老年代的大小也为10，其中新生代的大小设置为8：1，我们按顺序new4个对象，分别是2，2，2，4，那么当分配到第四个的时候，发现Eden区已经不足以放下4M大小的对象，那么会进行MiniorGC,就会把前面3个往老年代放，然后把4放在Eden区。

- 大对象直接进入老年代：所谓大对象指的是需要连续分配的内存空间大的对象称之为大对象，假如对象大于我们设定的大对象的阈值，那么对象会直接被分配到老年代去。

- 长期存活的对象进入老年代：对象如果经历的MiniorGC大于我们设定的阈值，那么对象会被直接分配给老年代。

#### 四、记录一次FullGC频繁的事情

前阵子部门使用了新的框架，发布之后随着时间的增长发现机器经常出现FullGC的现象。经过前面的例子已经知道了FullGC是在老年代的进行的GC操作。
反向推导：最终同事发现是缓存的key有问题，请求参数我们使用请求的queryTime跟一些其他的业务key作为缓存的key，本意是如果用户在几分钟之内做同样的请求的话可以帮服务器分担一些压力，降低RT。后来发现每次的queryTime都是不一样的，导致缓存的key是没有用的，每次都是取出新的数据并再次加入缓存，而且缓存的时间过长，冗余了非常多的无效的数据在内存中，导致了内存吃紧而引发FullGC。
  
  
 
   