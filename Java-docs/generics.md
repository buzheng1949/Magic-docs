#### 一、什么是泛型

开发中，经常会用到泛型，泛型是在JDK5之后的一个新特性，允许定义类、接口等的是实用类型参数。

泛型的出现，使得从需要在代码里面增加判断以及类型转化的事情交给编译器去解决，如下代码所示，当没有泛型的时候,代码如下：

```
	List nonGenerics = new ArrayList();
        nonGenerics.add("buzheng");
        Object nonGenericsObject = nonGenerics.get(0);
        if (nonGenericsObject != null) {
            //进行类型判断
            if (nonGenericsObject instanceof String) {
                //进行类型转换
                String s = (String) nonGenericsObject;
            }
        }
```
而当使用泛型的时候，可以看到的是我们的代码更加简洁：

```
 List<String> nonGenerics = new ArrayList();
 nonGenerics.add("buzheng");
 String s = nonGenerics.get(0);
```

这就是泛型带给我们代码的书写以及规范上的好处。

#### 二、泛型到底怎么玩的
首先先执行以下的代码，我们看看输出的效果:

```
	//类型参数为String
        List<String> stringList = new ArrayList<>();
        //类型参数为Integer
        List<Integer> integerList = new ArrayList<>();
        Class clazzStringList = stringList.getClass();
        Class clazzIntegerList = integerList.getClass();
        //看看两个类是否是共同的类
        System.out.println(clazzIntegerList == clazzStringList);
```

按照正常来讲，这里的输出应该是false，但是实际上是true，因此可以猜想，为什么类型居然一样了，明明类型参数都不一样的。

接下来为了验证我们的猜想，我们再通过如下代码进行验证。

```
/**
 * Created by buzheng on 17/12/8.
 */
public class TestGenerics {
    public static void main(String[] args) {
        try {
            //类型参数为String
            List<String> stringList = new ArrayList<>();
            stringList.add("hello");
            Class clazzStringList = stringList.getClass();
            Method method = clazzStringList.getMethod("add", Object.class);
            //加入一个int类型
            method.invoke(stringList, 1);
            System.out.println(stringList);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

可以看一下我们的输出结果：![输出结果](https://user-gold-cdn.xitu.io/2017/12/8/1603647b17589664?w=1163&h=300&f=png&s=26802)

可以看到的是，我们通过反射，在运行期给stringList加入类型为Integer类型的数值，是没有问题的。由此我们可以猜测在运行期的时候，作用在stringList上的类型参数是否可能被编译之后给去除掉了。

没错，如上面的两段代码所示：我们可以初步得出一个结论：**泛型在编译之后进行了类型擦除**

 - 什么是类型擦除？
 
 前面说到过泛型是在JDK5之后引入的，为了兼容5之前的代码，虚拟机会将我们使用的泛型进行擦除，还原为原始类型。所以,**Java的泛型是一种伪泛型，**以下面的代码为例子：
 
 ```
 /**
 * Created by buzheng on 17/12/8.
 */
public class TestGenerics {
    public static void main(String[] args) {
        List<String> stringList = new ArrayList();
        stringList.add("123");
        System.out.println(stringList.get(0));
    }
}
 ```
 
 我们反编译看下这个class文件编译之后变成什么样子？
 
 ![反编译class文件](https://user-gold-cdn.xitu.io/2017/12/8/1603647b1ba1f223?w=1106&h=708&f=png&s=60177)
 
 可以确确实实的看到，我们的类型确实是被擦除了，原先的String类型已经变成了Object类型。
 
 - 既然擦除了类型参数，为什么我们不需要强转？
 
 按照常理来讲，在class文件中，我们的类型信息已经被擦除了，那么返回的类型就是Object类型，但是为什么我们不需要进行强转就能得到正确的结果呢？
 
 在Java虚拟机中，class文件是以特殊格式存在的文件，我们查看一下字节码的文件信息，如下所示：
 
 ```
 Classfile /Users/buzheng/Desktop/TestGenerics.class
  Last modified 2017-12-8; size 882 bytes
  MD5 checksum 3f753fc7b9100702e8238181e9695a7e
  Compiled from "TestGenerics.java"
public class com.buzheng.generics.TestGenerics
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #11.#29        // java/lang/Object."<init>":()V
   #2 = Class              #30            // java/util/ArrayList
   #3 = Methodref          #2.#29         // java/util/ArrayList."<init>":()V
   #4 = String             #31            // 123
   #5 = InterfaceMethodref #32.#33        // java/util/List.add:(Ljava/lang/Object;)Z
   #6 = Fieldref           #34.#35        // java/lang/System.out:Ljava/io/PrintStream;
   #7 = InterfaceMethodref #32.#36        // java/util/List.get:(I)Ljava/lang/Object;
   #8 = Class              #37            // java/lang/String
   #9 = Methodref          #38.#39        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #10 = Class              #40            // com/buzheng/generics/TestGenerics
  #11 = Class              #41            // java/lang/Object
  #12 = Utf8               <init>
  #13 = Utf8               ()V
  #14 = Utf8               Code
  #15 = Utf8               LineNumberTable
  #16 = Utf8               LocalVariableTable
  #17 = Utf8               this
  #18 = Utf8               Lcom/buzheng/generics/TestGenerics;
  #19 = Utf8               main
  #20 = Utf8               ([Ljava/lang/String;)V
  #21 = Utf8               args
  #22 = Utf8               [Ljava/lang/String;
  #23 = Utf8               stringList
  #24 = Utf8               Ljava/util/List;
  #25 = Utf8               LocalVariableTypeTable
  #26 = Utf8               Ljava/util/List<Ljava/lang/String;>;
  #27 = Utf8               SourceFile
  #28 = Utf8               TestGenerics.java
  #29 = NameAndType        #12:#13        // "<init>":()V
  #30 = Utf8               java/util/ArrayList
  #31 = Utf8               123
  #32 = Class              #42            // java/util/List
  #33 = NameAndType        #43:#44        // add:(Ljava/lang/Object;)Z
  #34 = Class              #45            // java/lang/System
  #35 = NameAndType        #46:#47        // out:Ljava/io/PrintStream;
  #36 = NameAndType        #48:#49        // get:(I)Ljava/lang/Object;
  #37 = Utf8               java/lang/String
  #38 = Class              #50            // java/io/PrintStream
  #39 = NameAndType        #51:#52        // println:(Ljava/lang/String;)V
  #40 = Utf8               com/buzheng/generics/TestGenerics
  #41 = Utf8               java/lang/Object
  #42 = Utf8               java/util/List
  #43 = Utf8               add
  #44 = Utf8               (Ljava/lang/Object;)Z
  #45 = Utf8               java/lang/System
  #46 = Utf8               out
  #47 = Utf8               Ljava/io/PrintStream;
  #48 = Utf8               get
  #49 = Utf8               (I)Ljava/lang/Object;
  #50 = Utf8               java/io/PrintStream
  #51 = Utf8               println
  #52 = Utf8               (Ljava/lang/String;)V
{
  public com.buzheng.generics.TestGenerics();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 9: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/buzheng/generics/TestGenerics;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: new           #2                  // class java/util/ArrayList
         3: dup
         4: invokespecial #3                  // Method java/util/ArrayList."<init>":()V
         7: astore_1
         8: aload_1
         9: ldc           #4                  // String 123
        11: invokeinterface #5,  2            // InterfaceMethod java/util/List.add:(Ljava/lang/Object;)Z
        16: pop
        17: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
        20: aload_1
        21: iconst_0
        22: invokeinterface #7,  2            // InterfaceMethod java/util/List.get:(I)Ljava/lang/Object;
        27: checkcast     #8                  // class java/lang/String
        30: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        33: return
      LineNumberTable:
        line 11: 0
        line 12: 8
        line 13: 17
        line 14: 33
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      34     0  args   [Ljava/lang/String;
            8      26     1 stringList   Ljava/util/List;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            8      26     1 stringList   Ljava/util/List<Ljava/lang/String;>;
}
SourceFile: "TestGenerics.java
 ```
  在上述的字节码信息中，可以看到我们调用get方法的时候，确确实实是先返回了object类型
  
  ```
  22: invokeinterface #7,  2            // InterfaceMethod java/util/List.get:(I)Ljava/lang/Object;
  ```
  
  然后下面有一个checkcast，很明显这是一个判断是否需要进行类型转换的标志，checkcast指向了常量池的#8，而刚好对应的是
  
	```
	#8 = Class              #37            // java/lang/	String
	```
	
原来虚拟机在会先在编译完成的class文件中记录下泛型的信息，虽然编译后擦除了，但是记录在了字节码文件中，等到执行的时候会先checkcast进行类型转换，到此，泛型是如何进行擦除又能准确的帮助我们进行类型转换就很清晰了。
	
#### 三、最后的最后，类型擦除的过程是怎么样的优先级？
  - 首先找到替换类型参数的具体类，如果泛型没有extends任何的上限类，那么就是Object
  - 如果泛型extends了上限类，比如 T extends List，那么就是List了
  - 最后将找到的替换类替换掉泛型，生成信息或者桥接方法在字节码文件中
  
	
 
 
 
 
 

