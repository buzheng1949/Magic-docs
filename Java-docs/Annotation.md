#### 一、注解是什么

- 概念

  注解是一种元数据，一种用于描述数据的数据。如下面的代码所示：
  
  ```
  class Self {
        @Override
        public String toString() {
            return super.toString();
        }
    }
  ```
  Self类重写了toString()方法，该方法上有Override注解表明该方法是复写了父类的方法，如果我们写错了方法名，编译器会报错，这就是注解能帮我们做的事情。**从上面的例子我们可以看到注解就是用于描述数据的一种方式**
- 注解分类

 ```
 @Target(ElementType.METHOD)
 @Retention(RetentionPolicy.SOURCE)
 public @interface Override {
 }
 ```
 从上面的Override注解我们可以看到上面有一个@Retention的元注解，@Retention元注解表示该注解的保留期限。我们看下源代码如何定义@Retention。
 
 ```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
 ```
 可以看到注解的有效期限是根据RetentionPolicy这个枚举去确定的，我们可以看下RetentionPolicy是如何定义的。

	
	```
	public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
	}
	```
我们可以看到共3个有效时间，第一个是Source源码时保存、第二个CLASS表示会在class文件保存但是不会在VM或者运行期保存、第三个是RUNTIME表示从编译成class文件、VM或者RUNTIME时间都会进行保存。于是可以简单的将注解分为以下几类：
    1、源码注解
    2、编译注解
    3、运行注解
	
#### 二、注解的元注解解析

我们注意到注解经常出现的四个元注解,如下代码所示：

```
/**
 * Created by buzheng on 17/12/5.
 */
@Inherited
@Target(ElementType.METHOD)
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface PersonMessage {

    int age() default 18;

    String name() default "buzheng";
}
```
 - Inherited 注解可否被继承
 - Target 表示注解可以使用的目标位置，比较常见的是FIELD作用于字段，TYPE作用于类、接口、枚举等等，METHOD作用于方法等等，具体的如下：
 
 ```
 public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
 ```
 - Documented 注解可以生成文档
 - Retention 表示注解的有效持续时间，前面已经讲到了，可以分为以下三种
   - SOURCE 源码保留注解
   - CLASS 编译生成的class文件保留注解
   - RUNTIME 运行期保留注解
  
#### 三、如何写一个注解

- 注解用@interface关键字标识是一个注解
 
 ```
 public @interface PersonMessage {

 }
 ```
- 给注解加入元注解，一般加入上面我们说的四个元注解
 
	```
	@Inherited
	@Target(ElementType.METHOD)
	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	public @interface PersonMessage {
	
	}
	```
- 给注解定义字段
  - 注解支持多种字段类型，除了常用的基本数据类型外，还支持枚举、注解等等。
  - 注解字段默认值可加可不加
  - 如果注解只有一个字段，字段名必须为value
 
	 ```
	@Inherited
	@Target(ElementType.METHOD)
	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	public @interface PersonMessage {
	
	    int age() default 18;
	
	    String name() default "buzheng";
	}
	 ```

#### 四、如何使用并解析注解
假如我们定义了两个注解，一个是作用在类上面的，一个是作用在方法上面的，在运行的时候，我们需要取解析他们，代码如下所示：

```
package com.buzheng.annotation;

import java.lang.annotation.Annotation;
import java.lang.reflect.Method;

/**
 * Created by buzheng on 17/12/5.
 */
@TestAnnotation(name = "AnotationTest in class")
public class AnnotationTest {

    @PersonMessage(name = "yupeibiao", age = 23)
    private String getPersonMessage() {
        return "";
    }

    public static void main(String[] args) {
        AnnotationTest anotationTest = new AnnotationTest();
        Class c = anotationTest.getClass();
        //判断是否是这个注解
        boolean isAnotation = c.isAnnotationPresent(TestAnnotation.class);
        if (isAnotation) {
            //将类上的注解拿出来
            TestAnnotation testAnotation = (TestAnnotation) c.getAnnotation(TestAnnotation.class);
            //输出注解信息
            System.out.println(testAnotation.name());
        }

        Method[] methods = c.getDeclaredMethods();
        for (Method method : methods) {
            PersonMessage message = method.getAnnotation(PersonMessage.class);
            if (message == null) {
                continue;
            }
            System.out.println(message.age());
            System.out.println(message.name());
        }

        for (Method method : methods) {
            Annotation[] annotations = method.getAnnotations();
            for (Annotation annotation : annotations) {
                if (annotation instanceof PersonMessage) {
                    PersonMessage message = (PersonMessage) annotation;
                    System.out.println(message.age() + "  ");
                    System.out.println(message.name());
                }
            }
        }


    }

}
```

我们可以看到，无论是拿类上面的注解信息或者是方法上面的注解信息，我们都是通过反射的方法进行获取的，流程大概如下：

 -  通过反射获取需要解析的类
 -  如果需要获取类上面的注解，先判断类上面是否有注解c.isAnnotationPresent(YourAnnotation.class);
 -  获取类上的注解信息c.getAnnotation(YourAnnotation.class);
 -  如果需要获取方法或者字段的注解信息，大致流程跟上述获取类的注解信息的方法一致，只是调用的方法不同。
 
比如上面的流程我们可以运行获取到运行结果：
![运行结果](https://user-gold-cdn.xitu.io/2017/12/5/16024ef86c280bb0?w=1240&h=442&f=png&s=69384)

#### 五、简单的注解使用例子
 - 假如我们有这样的一个需求，我们有一个Query，需要跟SQL进行字段的一一对应（不考虑MyBatis），然后我们构建出一条SQL可以用于执行。那我们可以在Query类用注解指定类名，字段名，然后到时候通过反射获取每个字段的值，这样我们就能把整条SQL拼接起来
 
 首先我们在Query上需要使用的两个注解：
  - 第一个是定义在类上面，指定类对应表名的注解
  
   ```
   /**
	 * Created by buzheng on 17/12/5.
	 * 该注解用于类上，标明类对应的表名是什么
	 */
	@Inherited
	@Documented
	@Target(ElementType.TYPE)
	@Retention(RetentionPolicy.RUNTIME)
	public @interface Table {
	    String value();
	}

   ```
  - 第二个定义在类的字段上面，指定字段跟表的字段对应的注解
  
  ```
  /**
	 * Created by buzheng on 17/12/5.
	 * 该注解用于标明类字段跟表字段向的一一对应关系
	 */
	@Inherited
	@Target(ElementType.FIELD)
	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	public @interface Row {
	    String value();
	}
  ```
  - 给Query定义注解，绑定跟查询的表的一一对应关系
  
  ```
    /**
	 * Created by buzheng on 17/12/5.
	 */
	@Table("student")
	public class Query {

    @Row("student_name")
    private String name;

    @Row("student_age")
    private int age;

    @Row("student_grade")
    private int grade;

    @Row("student_number")
    private int number;

    @Row(("id"))
    private int id;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public int getGrade() {
        return grade;
    }

    public void setGrade(int grade) {
        this.grade = grade;
    }

    public int getNumber() {
        return number;
    }

    public void setNumber(int number) {
        this.number = number;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }
}

  ```
  
  - 解析注解、获取字段值并拼装SQL
  
  ```
	 /**
	 * Created by buzheng on 17/12/5.
	 */
	public class Test {
    public static void main(String[] args) {
        Query query = new Query();
        query.setAge(1);
        query.setId(1);
        query.setName("buzheng");
        System.out.println(getQueryFromAnnotation(query));
    }

    /**
     * 传进来查询的Query类 解析并拼装sql
     *
     * @param object
     * @return
     */
    public static String getQueryFromAnnotation(Object object) {
        Class clazz = object.getClass();
        boolean flag = clazz.isAnnotationPresent(Table.class);
        StringBuilder stringBuilder = new StringBuilder("select * from ");
        if (!flag) {
            return stringBuilder.toString();
        }
        Table table = (Table) clazz.getAnnotation(Table.class);
        stringBuilder.append(table.value());
        stringBuilder.append(" where 1=1 ");
        java.lang.reflect.Field[] fileds = clazz.getDeclaredFields();
        for (Field field : fileds) {
            boolean isFiledAnnotation = field.isAnnotationPresent(Row.class);
            if (isFiledAnnotation) {
                stringBuilder.append(" and ");
                Row row = (Row) field.getAnnotation(Row.class);
                String fieldName = field.getName();
                String methodName = "get" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1, fieldName.length());
                try {
                    Method method = clazz.getMethod(methodName);
                    Object value = method.invoke(object);
                    if (value == null || (value instanceof Integer && (Integer) value == 0)) {
                        continue;
                    }
                    stringBuilder.append(row.value() + " = " + value);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        return stringBuilder.toString();

    }
}
  ```
  
  运行上述代码可以看到我们的效果![代码运行效果](https://user-gold-cdn.xitu.io/2017/12/5/1602515aa08828e3?w=2490&h=452&f=png&s=82903)
  
 - 假如我们还有一个需求，我们需要用AOP对某些请求进行拦截，那我们也可以将注解作为切面，将需要拦截的方法增加注解的方式进行拦截，并且可以将注解的信息解析出来在切面进行使用。
 
 ```
	 /**
	 * Created by buzheng on 17/12/5.
	 */
	@Inherited
	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.METHOD)
	public @interface LogAnnotation {
	  /**
     * 操作
     * @return
     */
    String operateType() default "默认操作";

    /**
     * 操作人员工号ID
     * @return
     */
    int workId();
 }
 ```
 使用AOP的方式使用注解
 
 ```
 /**
 * Created by buzheng on 17/12/5.
 */
	@Aspect
	@Component
	public class MyAspect {
	
	    private Logger logger = LoggerFactory.getLogger(MyAspect.class);
	
	    @Pointcut("@annotation(com.buzheng.LogAnnotation)")
	    public void logPointCount() {
	    }
	
	    @Around("logPointCount()")
	    public Object actionLog(ProceedingJoinPoint joinPoint) {
	        Object res = new Object();
	        try {
	            //获取注解的各种信息
	            LogAnnotation logAnnotation = getAnnotation(joinPoint);
	            String operateType = logAnnotation.operateType();
	            int workId = logAnnotation.workId();
	            //然后你想做点什么，比如打个log
	            logger.warn("操作类型变更:" + operateType + "变更人工号:" + workId);
	            res = joinPoint.proceed();
	        } catch (Throwable throwable) {
	            logger.error("aop is failed", throwable);
	        }
	        return res;
	    }
	
	    /**
	     * 获取注解信息
	     *
	     * @param joinPoint
	     * @return
	     * @throws Exception
	     */
	    public LogAnnotation getAnnotation(ProceedingJoinPoint joinPoint) throws Exception {
	        Signature signature = joinPoint.getSignature();
	        MethodSignature methodSignature = (MethodSignature) signature;
	        Method method = methodSignature.getMethod();
	        boolean flag = method.isAnnotationPresent(LogAnnotation.class);
	        if (flag) {
	            LogAnnotation logAnnotation = (LogAnnotation) method.getAnnotation(LogAnnotation.class);
	            return logAnnotation;
	        }
	        return null;
	    }
	
	}
 ```
 
 
 