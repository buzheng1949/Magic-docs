#### Chapter 2. 核心技术

##### 依赖注入
 - Spring注入的几种方式
   - 通过XML显式配置
   - 通过Java显式配置
   - 自动装配
 - 自动装配
  - 创建可被发现的Bean，如下所示，bean被@Component注解，通过使用@ComponentScan注解或者在XML使用scan的方式进行自动装配
  
  ```java
  //声明组件
    @Component
public class JayChouCD implements CD {

    @Override
    public void play() {
        System.out.print("hello world");
    }
}
//通过Java进行扫描
@ComponentScan //自动扫描同个包下面的用component注解的bean (basePackages = {"aspect","charpter2"})
@Configuration
public class Config {
}
//通过xml扫描
<context:component-scan base-package:"com.buzheng"/>
``` 
  - 上述方式是将bean注入到容器中，相当于将bean放在一个bean工厂里面，而当你需要使用的时候，使用@AutoWired进行自动注入
  
- 一般情况下，如果bean是在工程自定义的，上述的方式可以满足主动装配，而当需要依赖的bean是第三方包的时候，就需要进行显式的配置。

 - 通过Java的方式进行显式配置
 
  1、扫描＋AutoWired的方式 直接通过@ComponentScan(basePackages = {"aspect","charpter2"})的方式指定扫描的包，然后使用AutoWired进行使用
  2、通过新增Java配置类，实际上Java配置类就是XML文件翻译成Java代码，使用姿势如下:
  
  ```java
  @Configuration
public class Config {

    @Bean(name = "jayChou") //这个名字是bean的名字 如果未注明 bean的名字会跟方法名一致
    public CD jayChouCD() {
        return new JayChouCD();
    }
}
  ```  
  想使用Java的方式配置bean，首先需要声明这个类是一个配置类，通过在类上方加入@Configuration注解，对应于XML文件上面的beans的一堆声明，@Bean注解会告诉Spring该方法需要返回一个对象，该对象需要注册为Spring应用上下文的bean，方法里面产生了bean的实体。
 - 通过XML的方式注册bean
  
  
  
   