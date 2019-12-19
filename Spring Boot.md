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
### 依赖注入  
在Java程序中，一个业务逻辑经常需要两个或两个以上的对象协作来完成，通常每个对象在使用他的合作对象时，自己均要使用像new object()这样的语法来完成合作对象的申请工作，这样会使得对象之间的耦合度较高，程序结构比较复杂。**而依赖注入的思想是：由Spring来负责控制对象的生命周期和对象间的关系，对象只需要负责关心业务逻辑就好了。**   
创建应用对象之间协作关系的行为通常称为装配(wiring),这也是依赖注入的本质。在Spring中依赖注入有多种方式，我们在这里先来介绍一下配置Spring容器最常见的三种方法。
#### 自动化装配bean
Spring从两个角度来实现自动化装配：  
- 组件扫描(component scanning):Spring会自动发现应用上下文中所创建的bean。
- 自动装配(autowiring):Spring自动实现bean之间的依赖。  
为了阐述组件扫描和装配，我们需要装配几个bean，它们代表一个音响系统中的组件。首先，创建一个CompactDisc接口和它的实现CompactDiscImp,并将其创建为一个bean。然后，会创建一个CDPlayer类，让Spring发现它，并将bean注入进来。
>CompactDisc接口在Java中定义了CD的概念
```java
public interface CompactDisc{
    void play();
}
```
我们将CompactDisc定义为一个接口，它定义了CD播放器的一系列操作，而且作为接口，它将CD的任意实现与CD本身的耦合降低到了最小的程度。这里，我们还需要一个CD的实现，也就是CompactDiscImp类。
>CompactDiscImp实现类
```JAVA
@Component
public class  CompactDiscImp implements CompactDisc{
    private String title = "这是实现类";
    public void play()
    {
        System.out.println(title);
    }
}
```
和接口一样，CompactDiscImp的具体内容并不重要，你需要注意的是CompactDiscImp类上使用了@Component注解。这个简单的注解表明该类会作为组件类，并告知Spring要为这个类创建bean。不过，组件扫描默认是不启用的。我们还需要显示配置Spring，从而命令它去寻找带有@Component注解的类，并为其创建bean。
>@ComponentScan注解启用了组件扫描（CDPlayerConfig.class）
```java
@Configurzation
@ComponentScan
public class CDPlayerConfig
{

}
```
类CDPlayerConfig通过Java代码定义了Spring的装配规则。在本文后部分，我们还会更为详细的介绍基于Java的Spring配置。不过，现在我们只需观察CDPlayerConfig类并没有显示地声明任何bean，只不过它使用了@ComponentScan注解，这个注解能够在Spring中启用组件扫描，使得Spring去寻找带有@ComponentScan注解地类，并为其创建bean。（此处也可以通过使用XML来启用组件扫描，但作者更喜欢使用JAVA配置，这里就不做赘述，如果有对XML配置感兴趣的，可以去查看Spring in Action的第二章）
>通过自动装配，将一个CompactDisc注入到CDPlayer之中
```JAVA
public class CDPlayer
{
    @Autowired//由于本实例只有一种实现类，所以默认注入对象为CompactDiscImp的实例，若有多个实现类，可用@Qualifier注解加以区分
    private CompactDisc cd;//不通过new对象的方式，而是通过注入的形式创建了一个其他类的对象

    cd.play();//会打印出“这是实现类”
}
```
这就实现了，自动装备bean的方式。
#### 通过Java代码装配bean
尽管在很多场景下通过组件扫描和自动装配实现Spring的自动化配置是更为推荐的方式，但有时候自动化配置的方案行不通，因此需要明确配置Spring。比如说，你想要将第三方库中的组件配到你的应用当中，这种情况你无法直接在第三方库的源码上添加@Component和@Autowired注解，因此就不能使用自动化装配的方案了。  
在这种情况下，你必须要采用显示装配的方式。在进行显示配置的时候，有两种方案可选：Java和XML。XML我在这里不作赘述，有兴趣的同学可以自行查阅《Spring in Action》第二章第四节，在这里我只介绍使用Java进行配置的方式。  
进行显示配置时，JavaConfig是更好的方案，因为他更为强大，类型安全而且对于重构比较友好，不同于其他的Java代码，JavaConfig是配置代码，它不应该含有任何业务逻辑，最好将其放在业务逻辑之外的包中。还是之前的例子。
>@ComponentScan注解启用了组件扫描（CDPlayerConfig.class）
```java
@Configurzation
@ComponentScan
public class CDPlayerConfig
{

}
```
创建JavaConfig类的关键在于为其添加@Configuration注解，@Configuration注解表明这个类是一个配置类。@ComponentScan启用了组件扫描，这是实现自动装配的关键。但如果我们将@ComponentScan注解移除，此时的@CDPlayerConfig类就不会有任何作用了，现在再运行主函数，会抛出BeanCreation-Exception异常。这是因为主函数期望被注入CompactDisc,但是这些bean根本就没有被创建，那我们如何进行显式的java配置呢？  
要在JavaConfig中声明bean，我们需要编写一个方法，这个方法**会创建所需类型的实例**，然后给这个方法添加@Bean注解。
>使用Javaconfig显式装配bean（CDPlayerConfig.class）
```java
@Configurzation
public class CDPlayerConfig
{

    @Bean
    public CompactDisc javaconfig()
    {
        return new CompactDiscImp();
    }

}
```
@Bean注解会告诉Spring这个方法会返回一个对象，该对象要注册为Spring应用上下文中的bean。
>通过Java装配，将一个CompactDisc注入到CDPlayer之中
```JAVA
public class CDPlayer
{
    @Autowired//由于本实例只有一种实现类，所以默认注入对象为CompactDiscImp的实例，若有多个实现类，可用@Qualifier注解加以区分
    private CompactDisc cd;//不通过new对象的方式，而是通过注入的形式创建了一个其他类的对象

    cd.play();//会打印出“这是实现类”
}
```