### 一、什么是 ClassLoader ？

- 什么是ClassLoader 
  
  当代码编写完成之后，我们会写很多的以Java文件，而这些Java文件会被编译生成class文件，class文件被需要被经历一系列的流程才能够进行使用与卸载。而在程序启动的时候，所有的class文件并不会一次性加载，而是在需要的时候，根据Java的类加载机制进行动态加载。而完成对class文件加载操作的，就是ClassLoader。
  
- ClassLoader的作用
  - 将类加载到内存中
  - 决定类是由哪个类加载器进行加载
  - 将class文件中的信息解析成对象
 
### 二、分析ClassLoader
- ClassLoader的类型
  - BootstrapClassLoader 
  - ExtClassLoader   
  - AppClassLoader  
 
 其中3个ClassLoader各自负责加载的区域不同，如下代码所示:
 
 ```
    /**
     * 测试classloader的加载范围
     */
    private void testClassLoader(){
        /**
         * BootstrapClassLoader 加载  jre/lib
         */
        System.out.println(System.getProperty("sun.boot.class.path"));

        /**
         * Extention ClassLoader   jre/lib/ext
         */
        System.out.println(System.getProperty("java.ext.dirs"));

        /**
         * AppClassLoader  bin目录
         */
        System.out.println(System.getProperty("java.class.path"));
    }
 ```
 
 通过以上代码，我们可以看到每个ClassLoader的加载范围是不一样的，其中BootStrapClassLoder加载的是jre/lib目录下的jar包，这些比如rat.jar等等。而ExtClassLoader负责加载的是jre/lib/ext目录下的jar包，而AppClassLoader主要加载的是我们定义的类的class文件。
 
- ClassLoader的加载机制
  ClassLoder的继承关系图如下所示：
  ![ClassLoader继承关系图](https://user-gold-cdn.xitu.io/2017/12/7/1603172695147f2f?w=591&h=552&f=png&s=29094)
  
  而当我们看到ClassLoder的源码的时候，我们可以看到：
  
  ```
  /**
     * Loads the class with the specified <a href="#name">binary name</a>.  The
     * default implementation of this method searches for classes in the
     * following order:
     *
     * <ol>
     *
     *   <li><p> Invoke {@link #findLoadedClass(String)} to check if the class
     *   has already been loaded.  </p></li>
     *
     *   <li><p> Invoke the {@link #loadClass(String) <tt>loadClass</tt>} method
     *   on the parent class loader.  If the parent is <tt>null</tt> the class
     *   loader built-in to the virtual machine is used, instead.  </p></li>
     *
     *   <li><p> Invoke the {@link #findClass(String)} method to find the
     *   class.  </p></li>
     *
     * </ol>
     *
     * <p> If the class was found using the above steps, and the
     * <tt>resolve</tt> flag is true, this method will then invoke the {@link
     * #resolveClass(Class)} method on the resulting <tt>Class</tt> object.
     *
     * <p> Subclasses of <tt>ClassLoader</tt> are encouraged to override {@link
     * #findClass(String)}, rather than this method.  </p>
     *
     * <p> Unless overridden, this method synchronizes on the result of
     * {@link #getClassLoadingLock <tt>getClassLoadingLock</tt>} method
     * during the entire class loading process.
     *
     * @param  name
     *         The <a href="#name">binary name</a> of the class
     *
     * @param  resolve
     *         If <tt>true</tt> then resolve the class
     *
     * @return  The resulting <tt>Class</tt> object
     *
     * @throws  ClassNotFoundException
     *          If the class could not be found
     */
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
  ```
  
  简单翻译一下类加载的过程：
  - 首先当需要加载类的时候，先检查需要加载的类是否已经加载。
  - 如果还没加载，自己不会去尝试加载类，而是将类加载的任务委托给了parentClass，向上进行类的查找过程。
  - 如果向上委托后，找到最顶上的BootStrapClassLoader类进行加载，如果没有加载到会交给下一级ExtClassLoader进行加载，如果在此从上到下查找的过程中查找到，则会将类加载到内存中，如果没有找到，会交给发起委托的类加载器，由它去指定的文件系统等进行查找加载，如果还是没有找到，则抛出ClassNotFound的异常。
  
  从上面可以看到的是，类的加载过程先由加载发起者进行向上委托，然后再由顶上的类加载器进行查找返回，找不到就自己加载的过程，这种加载的过程我们称作双亲委派机制。
  
  但是仔细一想，不对呀，假如AppClassLoader需要加载一个类的时候，会向上委托到ExtClassLoader，然后再向上委托给BootStrapClassLoader。如下代码所示：
  
  ```
   /**
     * 类加载过程
     */
    private static void parentDelegation(){
        ClassLoader classLoader = Test.class.getClassLoader(); //app
        System.out.println(classLoader);
        ClassLoader parentClassLoader = classLoader.getParent(); //ext
        System.out.println(parentClassLoader);
        ClassLoader parentParentClassLoader = parentClassLoader.getParent(); //null
        System.out.println(parentParentClassLoader);
    }
  ```
  
  ![执行结果](https://user-gold-cdn.xitu.io/2017/12/7/1603172695b5f873?w=839&h=150&f=png&s=27078)
  
  我们可以看到，输出结果除了null确实跟我们设想的一样，但是为什么BootStrapClassLoader为空呢，其实是因为BootStrapClassLoader是一个native的实现，所以在这里输出是null。但是仔细一想貌似还是不对，因为从我们上面给出的ClassLoader继承关系图，我们看到这几个ClassLoader没有继承关系，那为啥叫双亲委派，从代码中我们可以看到这个“亲”的定义：
  
  ```
   protected ClassLoader(ClassLoader parent) {
        this(checkCreateClassLoader(), parent);
    }
  ```
  原来，是在产生一个类加载器的时候，通过传入parent指定了该ClassLoader的亲是哪个ClassLoader。它们之前是没有任何的继承关系的。
  
- 双亲委派的作用是什么？

   从上面可以看到，ClassLoader采用的是双亲委派机制，那到底有什么作用呢？
 - 减少重复加载
   
   在双亲委派机制中，当它的parentClassLoader已经加载了该类的时候，就没必要自己再加载一遍了，可以减少重复加载的可能性。
   
  - 安全
  
     假如没有双亲委派机制，那我们就可以随意让自定义的ClassLoader去加载经过改造的系统的类，比如String，这是一件非常危险的事情，而有双亲委派的时候，因为String类已经被加载过，所以就不能被我们自定义的ClassLoader加载。
   
- ClassLoder常见方法
  -  Class<?> defineClass(byte[] b, int off, int len) 表示将一个byte数组中的内容转换成一个类的实例。
  -  Class<?> findClass(String name) 从指定的位置查找类，返回一个类的实例。（自定义类加载器主要实现该方法）
  -  Class<?> loadClass(String name) 从指定的位置加载类。
   
### 三、实现自定义ClassLoader
 前面已经描述了ClassLoader的作用以及加载的过程，而当我们需要自定义一个ClassLoder去加载我们自己的类并执行某些操作的时候，就需要实现自定义ClassLoder。

- 首先需要继承ClassLoader类
- 实现自定义的findClass方法

以下代码为例：

```
/**
 * Created by buzheng on 17/12/7.
 */
public class TestClassLoader {

    public static void main(String[] args) {
        try {
            MyClassLoder myClassLoder = new MyClassLoder();
            Class clazz = myClassLoder.loadClass("com.buzheng.class_test.TestClassLoader");
            Object object = clazz.newInstance();
            Method method = clazz.getMethod("say", null);
            Object o = method.invoke(object, null);
            System.out.println(o);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    /**
     * 定义一个方法
     *
     * @return
     */
    public String say() {
        return "here";
    }


    static class MyClassLoder extends ClassLoader {
        @Override
        protected Class<?> findClass(String name) throws ClassNotFoundException {
            try {
                String classPath = name.substring(name.lastIndexOf(".") + 1) + ".class";
                InputStream is = getClass().getResourceAsStream(classPath);
                if (is == null) {
                    return super.findClass(name);
                }
                byte[] b = new byte[is.available()];
                return defineClass(name, b, 0, b.length);

            } catch (IOException e) {
                return super.findClass(name);
            }


        }
    }

}
```

执行main方法可以看到，我们确实自定义了一个类加载器，并且将字节码文件转为字节流进行自定义加载。
![执行结果](https://user-gold-cdn.xitu.io/2017/12/7/16031726989504d1?w=785&h=148&f=png&s=18372)


   

 


