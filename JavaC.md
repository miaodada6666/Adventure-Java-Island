
# 浅入浅出Javac编译原理


Java语言是当今程序员中使用最广的语言，不光是从语言本身来说，还包括了与Java相关的一些概念。例如JDK，J2EE，JVM等等。还不断有新的语言出现，如groove，scale等，他们到底和Java有什么关系，为什么这些非Java语言也能够运行在JVM上？Java又和JVM有什么关系呢？
今天这篇文章就是来探索这个问题的，Java语言有Java语言规范，而Java虚拟机也有Java虚拟机规范。二者都有自己的词法和语法解析规则，那么如何才能让Java的语法规则适应Java虚拟机的语法规则呢？这个任务就是由Javac编译器来实现的，他的任务就是将Java语言规范转化为Java虚拟机语言规范，将Java的源代码转化为class字节码。小编认为，掌握任何一门语言的前提都是要明白语言底层的编译机制，所以我们爪哇岛的第一篇文章就从JavaC的编译原理开始！  


**目录**
<!-- TOC -->

- [浅入浅出Javac编译原理](#浅入浅出javac编译原理)
    - [1.Javac是什么?](#1javac是什么)
    - [2.Javac编译器的基本结构](#2javac编译器的基本结构)
    - [3.设计模式之访问者模式](#3设计模式之访问者模式)
        - [3.1访问者模式基本介绍](#31访问者模式基本介绍)
        - [3.2访问者模式实现](#32访问者模式实现)
            - [表示电脑元素的接口————ComputerPart.java](#表示电脑元素的接口computerpartjava)
            - [创建实现上述接口的实现类———Keyboard.java Monitor.java和Mouse.java](#创建实现上述接口的实现类keyboardjava-monitorjava和mousejava)
            - [定义一个表示访问者的接口 ——— ComputerPartVisitor.java](#定义一个表示访问者的接口--computerpartvisitorjava)
            - [创建实现了上述接口的实体访问者———ComputerPartDisplayVisitor.java](#创建实现了上述接口的实体访问者computerpartdisplayvisitorjava)
            - [使用Main函数来显示Computer的组成部分](#使用main函数来显示computer的组成部分)
            - [执行程序，输出结果](#执行程序输出结果)
        - [3.3 Javac中访问者模式的实现](#33-javac中访问者模式的实现)

<!-- /TOC -->

@[toc]
## 1.Javac是什么?
Javac是一种编译器，能够将一种语言规范转化为另外一种语言规范。 
- 对于C,C++,汇编语言等语言采用的是一边编译一边执行的方式。这些语言可以将源码直接编译为CPU可以识别的目标机器码,因此执行时占用资源较少而且编译速度较快。编译器在这里的功能就是将语言规范转化为机器码规范。
- 对于Java语言来说，由于引入了Java虚拟机，不能够将源码直接编译为CPU可以识别的机器码，因此采用的是完全编译之后才能执行，所以占用的时间和空间都比较大。编译器（Javac）在这里的功能就是将Java源代码转化为Java虚拟机所能够识别的JVM语言，Java虚拟机再进一步将JVM语言编译为CPU可以识别的目标机器码。
## 2.Javac编译器的基本结构
&emsp;&emsp;想要搞清楚Javac编译器的基本结构，那么首先就要明白一个编译器将一种语言规范转化为另外一个语言规范需要经过哪些步骤？这就要想起大学时编译原理这门课的知识了。  
&emsp;&emsp;首先，要读取源码，一个字节一个字节的都进来，找出字节中有哪些是我们定义的语法关键词，如Java中的If ，while ，for等词语，还要识别哪些关键字是合法的哪些不是，这个步骤就是词法分析过程。词法分析的结果就是形成一个符合Java规范的Token流，就像是在日常和朋友交流中，朋友告诉你一句话，你要能分辨出哪些是标点符号，哪些是主语，哪些是谓语等等。  
&emsp;&emsp;接下来就是要对这些token流进行语法分析了，这一步就是检查这些关键词在一起是不是符合Java语法规范，比如If关键词后边跟的是不是布尔表达式。就像人类语言当中是不是有主谓宾，主谓宾的结合是不是正确，比如吃这个关键词后边就不能跟”屎“这个词语。语法分析的结果是形成一个符合Java规范的抽象语法树。抽象语法树是一个结构化的语法表达形式，它的作用是把语言的主要词法用一个结构化的形式组织在一起，对于这棵语法树我们可以在后面按照新的规则再重新组织，这也是编译器非常重要的功能之一。  
&emsp;&emsp;第三步是语义分析，语义分析是把一些难懂的复杂的语法转化为更加简单的易于编译器理解的语法，例如将for each转化为for循环结构，解释注解等等。这个步骤对应到人类语言中类似于将文言文转化为大家都能懂的白话文。语义分析的结果是形成一个新的抽象语法树，这个语法树更加接近JVM语言的语法规则。  
&emsp;&emsp;最后一步，通过字节码生成器根据新的抽象语法树生成字节码，也就是将一个数据结构（抽象语法树）转化为另一个数据结构（字节码）。就像所有的中文词语翻译成英文单词之后，按照英文语法组装成英文语句。  
&emsp;&emsp;代码生成器的结果就是生成符合Java虚拟机规范的字节码了。这个过程需要的组件可以参照下图。  
>Javac组件
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL3dpa2kvbWlhb2RhZGE2NjY2L3BpY3R1cmVzL0phdmFjX2NvbXBvbmVudC5wbmc?x-oss-process=image/format,png)


&emsp;&emsp;Javac的各个模块完成了将Java源代码转化成JVM字节码的任务。Javac主要有四个模块，分别是词法分析器，语法分析器，语义分析器和代码生成器。

## 3.设计模式之访问者模式
&emsp;&emsp;前面介绍的词法分析器、语法分析器、语义分析器和代码生成器中有多次遍历语法树的过程，然而每次遍历这棵语法树都会进行不同的处理动作，对这棵语法树也要进行进一步的处理。这是如何实现的呢？这实际上就是采用了访问者模式设计的，每次遍历都是一次访问者的执行过程。
### 3.1访问者模式基本介绍
&emsp;&emsp;在访问者模式中，我们使用了一个访问者类，目的是将数据结构与数据操作相分离，降低稳定的数据结构和易变的数据操作之间的耦合性。在被访问的类里面加一个和对外提供接待访问者的接口，将自身引用传入访问者。  
&emsp;&emsp;优点：符合单一职责原则；优秀的扩展性，可以方便满足不同的访问需求；访问条件的灵活性。
&emsp;&emsp;缺点：具体元素对访问者公布细节，违反了迪米特原则；具体元素变更比较困难，需重写相应的接口；违反了依赖倒置原则，没有依赖抽象，依赖了具体类！  
&emsp;&emsp;建议使用场景：对象结构中类很少改变，但是需要经常在对象结构上定义新的操作；需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免这些操作”污染“这些对象。
### 3.2访问者模式实现
>访问者模式实现UML图
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL3dpa2kvbWlhb2RhZGE2NjY2L3BpY3R1cmVzL0phdmFjX3Zpc2l0b3IucG5n?x-oss-process=image/format,png)
#### 表示电脑元素的接口————ComputerPart.java
```java
public interface ComputerPart {
   public void accept(ComputerPartVisitor computerPartVisitor);
}
```
#### 创建实现上述接口的实现类———Keyboard.java Monitor.java和Mouse.java
```java
public class Keyboard  implements ComputerPart {
 
   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
       //暴漏了this，使得该类可以被visit
      computerPartVisitor.visit(this);
   }
}
```
```java
public class Monitor  implements ComputerPart {
 
   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      computerPartVisitor.visit(this);
   }
}
```
```java
public class Mouse  implements ComputerPart {
 
   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      computerPartVisitor.visit(this);
   }
}
```
####  定义一个表示访问者的接口 ——— ComputerPartVisitor.java 
```java
public interface ComputerPartVisitor {
   public void visit(Mouse mouse);
   public void visit(Keyboard keyboard);
   public void visit(Monitor monitor);
}
```
#### 创建实现了上述接口的实体访问者———ComputerPartDisplayVisitor.java
```java
public class ComputerPartDisplayVisitor implements ComputerPartVisitor {
   @Override
   public void visit(Mouse mouse) {
       //add some codes to consume the elements of Class mouse
      System.out.println("Displaying Mouse.");
   }
 
   @Override
   public void visit(Keyboard keyboard) {
        //add some codes to consume the elements of Class keyboard
      System.out.println("Displaying Keyboard.");
   }
 
   @Override
   public void visit(Monitor monitor) {
        //add some codes to consume the elements of Class monitor
      System.out.println("Displaying Monitor.");
   }
}
```
#### 使用Main函数来显示Computer的组成部分
```java
public class VisitorPatternDemo {
   public static void main(String[] args) {
 
      ComputerPart computer = new Computer();
      computer.accept(new ComputerPartDisplayVisitor());
   }
}
```

#### 执行程序，输出结果
```java
Displaying Mouse.
Displaying Keyboard.
Displaying Monitor.
```

&emsp;&emsp;访问者模式中一般有抽象访问者、具体访问者、抽象节点元素、具体节点元素、结构对象和客户端几种角色，它们的具体左右如下所述。

- 抽象访问者（ComputerPartVisitor）：声明所有访问者需要的接口.
- 具体访问者（ComputerPartDisplayVisitor）：实现抽象访问者声明的接口。
- 抽象节点元素（ComouterPart）：提供一个接口，能够接受访问者作为参数传递给节点元素。
- 具体节点元素（Keyboard，Mouse和Monitor）：实现抽象节点元素声明的接口。
- 结构对象（本样例中没有）：提供一个接口，能够访问到所有的节点元素，一般作为一个集合特有节点元素的引用。
- 客户端（VisitorPatternDemo）：创建节点元素的对象，调用访问者访问节点元素。

### 3.3 Javac中访问者模式的实现
&emsp;&emsp;访问者模式可以将数据结构和对数据结构的操作解耦，使得增加对数据结构的操作不需要去修改数据结构，也不必去修改原有的操作，而是在执行时再定义新的具体访问者（visitor）就行了。在Javac中，不同的编译阶段都定义了不同的访问者模式实现。Javac就是基于访问者模式来遍历语法树的，在编译的不同阶段（词法分析，语法分析，语义分析，字节码生成器）定义不同的访问者从而实现在不需要修改原有语法树的数据结构的前提下，多次访问语法树

参考教程：  
菜鸟教程——[访问者模式](https://www.runoob.com/design-pattern/visitor-pattern.html)  
许令波——[深入分析Java Web技术内幕（第四章）](https://www.jianshu.com/p/f652d5f4ccd0)
