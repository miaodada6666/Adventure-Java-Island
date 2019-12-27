# 浅学Spring Boot
Spring Boot是Spring的扩展，在保持了Spring原有两大特性的前提下又提出了新的功能。
因此，正式介绍Spring Boot之前，我们不妨先来探寻一下Spring是个什么东西！
<!-- TOC -->

- [浅学Spring Boot](#浅学spring-boot)
    - [Spring](#spring)
        - [面向切面编程(AOP)](#面向切面编程aop)
        - [依赖注入](#依赖注入)
            - [自动化装配bean](#自动化装配bean)
            - [通过Java代码装配bean](#通过java代码装配bean)
    - [Spring Boot](#spring-boot)
        - [自动配置](#自动配置)
        - [起步依赖](#起步依赖)
        - [命令行界面](#命令行界面)
        - [Actuator](#actuator)

<!-- /TOC -->
## Spring
框架开发的初心都是为了降低程序员写代码时的复杂度，能够让程序员将自己的精力尽可能多的集中到自己的逻辑开发上面，而不是在开发环境的配置上面。为了实现这一点，Spring提出了以下两个技术：面向切面编程和依赖注入。
### 面向切面编程(AOP)
面向切面编程，通俗来说就是把专业的活交给专业的人来干。举个例子，就像在学校里有很多食堂，这些食堂都被不同的餐饮公司所承包，这里被承包了的食堂就是切面。这样做有什么好处呢，很明显，食堂承包出去之后由于餐饮公司是专门做餐饮的，那饭菜肯定比学校自己做的好吃，而且学校将餐饮业务承包出去之后，就可以将自己更多的精力放在教书育人上边，专注于自己的本职业务，如果对承包商不满意还可以即时的更换其他承包商（更换切面），真是个一举多得的好事情，在编程当中，什么是这些切面呢？对象与对象之间，方法与方法之间，模块与模块之间都是一个个这样的切面。
>单一职责原则：类应该是纯净的，不应含有与本身无关的逻辑。
面向切面编程，可以带来代码的解耦，使得各个模块专注于自身的业务逻辑，提高开发效率。
使用AOP之前，我们需要理解几个概念。  

![](http://q1mbfn418.bkt.clouddn.com/Spring%20Boot%20AOP.png)

传送门：[Spring 之 AOP](https://juejin.im/post/5a3201ff6fb9a0450f21f3ea)
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
>通过Java装配，将一个CompactDisc注入到CDPlayer之中。
```JAVA
public class CDPlayer
{
    @Autowired//由于本实例只有一种实现类，所以默认注入对象为CompactDiscImp的实例，若有多个实现类，可用@Qualifier注解加以区分
    private CompactDisc cd;//不通过new对象的方式，而是通过注入的形式创建了一个其他类的对象

    cd.play();//会打印出“这是实现类”
}
```
## Spring Boot
Spring Boot是Spring的扩展，在Spring的AOP(面向切面编程)和DI(依赖注入)两个特性的基础上又完善了以下四个功能：
- 自动配置：针对很多Spring应用程序所常见的应用功能，Spring boot能自动提供相关配置。  
- 起步依赖：告诉Spring Boot需要什么功能，它就能引入需要的库。
- 命令行界面：这是Spring Boot的可选特性，借此你只需写代码就能完成完整的应用程序，无需传统项目构建。
- Actuator：让你能够深入运行中的Spring Boot应用程序，一探究竟。 
### 自动配置
在任何Spring应用的源码中，你都会找到Java配置或XML配置（只使用自动化配置难以配置第三方库中的组件），它们为应用程序开启了特定的特性和功能。举个例子，如果你写过用JDBC访问关系型数据库的应用程序，那你一定在Spring应用程序上下文里配置过JdbcTemplate这个Bean。
```JAVA
@Bean
public JdbcTemplate jdbcTemplate(DataSource dataSource){
    return new JdbcTemplate(dataSource);
}
```
这段非常简单的Bean声明创建了一个JdbcTemplate的实例，注入了一个DataSource依赖，这意味着你还需要配置一个DataSource的Bean，这样才能满足依赖。  
虽然这两个Bean配置方法都不复杂，也不是很长，但它们只是Spring应用程序配置的一小部分。除此之外，还有无数Sping应用程序有着类似的配置方法，简而言之，这就是一个样板配置。  
既然它如此常见，那我们有没有办法可以自动配置这些Bean而不是每次手动去写配置代码呢？  
Spring Boot做到了这点，它会为这些常见配置场景进行自动配置。如果Spring Boot在应用程序的ClassPath里发现了JdbcTemplate，那么它会为你配置一个JdbcTemplate的Bean，如果在应用程序的Classpath里发现了DataSource,那么他还会为你创建一个DataSource的Bean。你无需担心那些Bean的配置，Spring Boot会做好准备，随时都能将其注入到你的Bean当中。这就是Spring Boot的自动配置.

### 起步依赖
Spring boot通过起步依赖为项目的依赖管理提供帮助。起步依赖就是特殊的Maven依赖和Gradle依赖，利用传递依赖解析，帮常用库聚合在一起。举个例子，假如你正在用Spring MVC构造REST API，并使用JSON作为资源表述。此外，你还想使用JSR-303规范的声明式校验，并使用嵌入式Tomcat服务器来提供服务。要实现以上目标，你在Maven或Gradle里至少需要以下8个依赖：
>org.springframework:spring-core  
>org.springframework:spring-web  
>org.springframework:spring-webmvc  
>com.fasterxml.jackson.core:jackson-databind  
>org.hibernate:hibernate-validator  
>org.apache.tomcat.embed:tomcat-embed-core  
>org.apache.tomcat.embed:tomcat-embed-el  
>org.apache.tomcat.embed:tomcat-embed-logging-juli  
如果打算使用Spring Boot的起步依赖，你只需要添加Spring Boot的Web起步依赖（org.springframework.boot:spring-boot-starter-web），仅此一个。它会根据依赖传递把其他所需依赖引入项目里。  
比起减少依赖数量，起步依赖还有一个好处就是：你不再需要考虑支持某种功能需要使用什么库了，直接引入相关依赖就行。如果应用是个Web应用，所以加入了Web起步依赖；如果应用程序要用到JPA持久化，那么就可以加入JPA起步依赖；需要安全功能，直接加入security起步依赖即可。
### 命令行界面
除了自动配置和起步依赖，Spring Boot还提供了一种很有意思的开发spring boot应用的新方法。Spring Boot Cli让只写代码即可实现应用程序成为可能。  
Spring Boot Cli利用了起步依赖和自动配置，让你专注于代码本身。简单来说，CLI能检测到你使用了哪些类，它知道要向Classpath中添加哪些起步依赖才能让它运转起来，一旦那些依赖出现在Classpath中，一系列自动配置就会接踵而来。  
但同时，Spring Boot Cli是Spring Boot的非必要组成部分，虽然它为Spring带来了惊人的力量，大大简化了开发，但也引入了一套不太常规的开发模型。如果您还是不太适应这种开发模型，可以抛开CLI，利用Spring Boot提供的其他东西。
### Actuator
Spring Boot的最后一个法宝就是Actuator，其他几个部分旨在简化Spring配置，而Actuator则要在提供运行时检查应用程序内部情况的能力。包括以下细节:
- Spring应用程序上下文配置的Bean。
- 应用程序取到的环境变量、系统属性、配置属性和命令行参数。
- 应用程序中线程的当前状态。
- 应用程序最近处理过的Http请求追踪情况。
- 各种和内存用量，垃圾回收、Web请求相关的指标。

传送门，[使用Spring Boot Actuator监控应用](http://www.ityouknow.com/springboot/2018/02/06/spring-boot-actuator.html)