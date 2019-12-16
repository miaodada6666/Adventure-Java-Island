# 浅学Spring Boot
Spring Boot是Spring的扩展，在Spring的AOP(面向切面编程)和DI(依赖注入)两个特性的基础上又完善了以下四个功能。  
- 自动配置：针对很多Spring应用程序所常见的应用功能，Spring boot能自动提供相关配置。  
- 起步依赖：告诉Spring Boot需要什么功能，它就能引入需要的库。
- 命令行界面：这是Spring Boot的可选特性，借此你只需写代码就能完成完整的应用程序，无需传统项目构建。
- Actuator：让你能够深入运行中的Spring Boot应用程序，一探究竟。  
因此，正式介绍Spring Boot之前，我们不妨先来探寻以下Spring是个什么东西！
## Spring
框架开发的初心都是为了降低程序员写代码时的复杂度，能够让程序员将自己的精力尽可能多的集中到自己的逻辑开发上面，而不是在开发环境的配置上面。为了实现这一点，Spring提出了以下两个技术：面向切面编程和依赖注入。
### 面向切面编程(AOP)
面向切面编程，通俗来说就是把专业的活交给专业的人来干。举个例子，就像在学校里有很多食堂，这些食堂都被不同的餐饮公司所承包，这里被承包了的食堂就是切面。这样做有什么好处呢，很明显，食堂承包出去之后由于餐饮公司是专门做餐饮的，那饭菜肯定比学校自己做的好吃，而且学校将餐饮业务承包出去之后，就可以将自己更多的精力放在教书育人的上边，专注于自己的本职业务，如果对承包商不满意还可以即时的更换其他承包商，真是个一举多得的好事情，在编程当中，什么是这些切面呢？对象与对象之间，方法与方法之间，模块与模块之间都是一个个这样的切面。
>单一职责原则：类应该是纯净的，不应含有与本身无关的逻辑。
面向切面编程，可以带来代码的解耦，使得各个模块专注于自身的业务逻辑，提高开发效率。
使用AOP之前，我们需要理解几个概念。

图

-  连接点(Join Point):  
所有**可能**的需要注入切面的地方。比如方法前后，类初始化、属性初始化前后。  
- 切点(Poincut):  
需要做某些处理的连接点（连接点有很多，但是真正切入的连接点才叫切点），比如打印日志、处理缓存等，存在于切面当中。目前，Spring只支持方法的切点定义。  
- 通知(Advice):  
定义在什么时候，做什么事情。Spring支持5种方法上的通知类型。   
@Before:在目标方法被调用之前调用。  
@After：通知方法会在目标方法返回或抛出异常之后调用。    
@AfterReturning：通知方法会在目标方法返回后调用。  
@AfterThrowing：通知方法会在目标方法抛出异常之后调用。  
@Around：相当于Befor和After的结合。
- 切面(Aspect)  
通知+切点的集合，定义在什么地方什么时间做什么事情。
```JAVA
@Aspect
public class Audience {
    @Autowired
    private CriticismEngineImpl criticismEngine;
    @Pointcut(
            "execution(* com.example.aoptest.impl.Performance.perform(int))"+"&& args(ha)")//将perform函数视为切点
    public void performance(int ha) {}//该方法本身只是个标识，供Advice依附，对performance函数的通知相当于对切点perform函数的通知。
    @Before("performance(ha)")//通知
   public    void silenceCellPhones(int ha)
    {
        ha++;
        System.out.println(ha);
        System.out.println("前置通知：Sillencing cell phones");
    }
}
```
- 引入(Introduction)  
允许我们向现有的类添加新方法或属性。就是把切面用到目标类当中，在不修改目标类的条件下为目标类添加新的属性和方法。