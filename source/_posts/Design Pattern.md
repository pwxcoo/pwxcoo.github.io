---
title: Design Pattern
date: 2018-06-23 16:25:24
categories: base
tags:
- base
- design pattern
---

# 创建型
## 单例模式 (Singleton) 

使用一个私有构造函数，一个私有静态变量以及一个公有静态函数来实现。

私有构造函数保证了不能通过构造函数来创建对象实例，只能通过公有静态函数返回唯一的私有静态变量。

```java
public class Singleton
{
    private static Singleton uniqueInstance;

    private Singleton(){}

    public static Singleton getUniqueInstance(){}
}
```

### 懒汉式 —— 线程不安全

```java
public class Singleton {
    private static Singleton uniqueInstance;

    private Singleton(){}

    public static Singleton getUniqueInstance() {
        if(uniqueInstance == null)
        {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
```

### 懒汉式 —— 线程安全

`synchronized` 保证在该静态方法只有一个线程能访问，但是性能有问题。
```java
public static synchronized Singleton getUniqueInstance() {
    if(uniqueInstance == null)
    {
        uniqueInstance = new Singleton();
    }
    return uniqueInstance;
}
```

### 饿汉式 —— 线程安全

```java
private static Singleton uniqueInstance = new Singleton();
```

### 双重校验锁

在实例化部分代码加锁。

在实例化过程中，仍然有线程可以通过第一个 `if` 判断，所以在加锁代码里再加一层判断。
```java
public class Singleton {
    private volatile static Singleton uniqueInstance;

    private Singleton(){}

    public static Singleton getUniqueInstance() {
        if(uniqueInstance == null) {
            synchronized(Singleton.class)
            {
                if(uniqueInstance == null)
                {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

### 枚举实现

单例模式的最佳实践。

可以防御反射攻击 (SetAccessible() 方法) 和序列化时生成新的实例 (防止序列化生成新的实例，必须声明所有字段都是 `transient`，并且提供一个 readResolve() 方法) 。
```java
public enum Singleton {
    uniqueInstance;
}
```
### References about Singleton
- [CyC2018/Interview-Notebook](https://github.com/CyC2018/Interview-Notebook/blob/master/notes/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.md#1-%E5%8D%95%E4%BE%8Bsingleton)
- [为什么要用枚举实现单例模式 (避免反射、序列化问题) ](https://www.cnblogs.com/chiclee/p/9097772.html)

## 简单工厂 (Simple Factory)

将客户类和具体子类实现解耦。
```java
public interface Product {
}

public class ConcreteProduct implements Product {
}

public class ConcreteProduct1 implements Product {
}

public class ConcreteProduct2 implements Product {
}

public class SimpleFactory {
    public Product createProduct(int type) {
        if (type == 1) {
            return new ConcreteProduct1();
        } else if (type == 2) {
            return new ConcreteProduct2();
        }
        return new ConcreteProduct();
    }
}

public class Client {
    public static void main(String[] args) {
        SimpleFactory simpleFactory = new SimpleFactory();
        Product product = simpleFactory.createProduct(1);
    }
}
```

## 工厂模式 (Factory Pattern)

当明确不同条件下创建不同实例的时候，使用工厂模式。

就是把具体要实例化什么类给在封装一层。相较简单工厂，把实例化推迟到了子类。

- 日志记录器。记录到本地磁盘，Console，远程服务器。
- 数据库访问。不同数据库。
- 连接服务器的框架。“POP3”，“IMAP”，“HTTP”


## 抽象工厂(Abstract Factory)

工厂模式会生成一个多余的工厂类，很多的工厂类就要在抽象一层，抽象工厂就是工厂的工厂。


## 建造者模式 (Builder Pattern)

封装一个对象的构造过程。因为构造很复杂，所以封装一层，方便调用。如一份套餐，需要构造一堆类，将构造过程封装。

## 原型模式 (Prototype Pattern)

用于创建重复的对象。适用于当创建一个新的对象成本较大时。

1. 在 Java 中用 Cloneable 接口，重写 clone() 实现浅拷贝
2. 用序列化等方式实现深拷贝

---

# 行为型
## 责任链 (Chain Of Responsibility)
使多个对象都有机会处理请求，让这些对象连成一条链。

Logger，不同的日志级别。是自己级别就打印，不是就传给下一个记录器。Fileter。

## 命令 (Command)
就是把行为请求者和行为实现者分开。我感觉很像消息队列的模式。。进消息，然后取出消息，然后调用命令运行。

三个角色
1. receiver
2. Command
3. invoke

好处是降低了系统的耦合度。一些事务延迟执行，或者 undo，redo 操作都能用命令模式。

## 解释器 (Interpreter)
解释一个特定的上下文。用在 SQL 解析，编译器等。

就是用给定的文法解析一个 Context。

- [实现样例1](https://github.com/CyC2018/Interview-Notebook/blob/master/notes/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.md#3-%E8%A7%A3%E9%87%8A%E5%99%A8interpreter)
- [实现样例2](http://www.runoob.com/design-pattern/interpreter-pattern.html)

## 迭代器 (Iterator) 
迭代器。

## 中介者 (Mediator)
降低多个类和对象之间的通信复杂度。MVC 模型中的 Controller 就是中介者。

缺点是中介者会变得庞大和难以维护。

## 备忘录 (Memento)
存档，`ctrl + z`都是备忘录模式。

客户不与备忘录类耦合，备忘录管理类和备忘录类耦合。客户只能看到备忘录管理类的接口，看不到备忘录的接口。不破坏封装性。

实现可以把备忘录类作为备忘录管理类的内部类。

## 观察者 (Observer)
当对象存在一对多的依赖关系，然后对象状态改变就会通知到所有依赖他的对象。

大概是这么一种关系: 
1. Subject
2. Observer抽象
3. Client extends Observer

Subject 改变状态给 Client 发送通知。

## 状态模式 (State)
类的行为是基于状态而改变的。

如果不用状态模式，会产生大量的 if, else 语句块。


用一个酒店入住的例子: 
```java
class State{}
class FreeState extends State{}
class BookedState extends State{}
class CheckInState extends State{}

public room{
    State nowState;
    void book{
        nowState.book();
    }
}
```

缺点是会让类变得很多。

## 策略模式 (Strategy)
让一个类的行为和算法中运行时可以改变。

状态注重的状态的改变，外界关心的是提供的接口。策略注重于外部的环境，根据外部的条件选择不同的策略。

## 模板方法 (Template)
放一些通用算法抽象出来。一些方法通用，为每个子类重写这些方法。

行为由父类控制，子类实现。为防止恶意操作，一般模板方法都加上了 final 关键字。

## 访问者模式 (Visitor) 
数据结构稳定。但是元素执行算法随着访问者改变而改变。

如 OA 系统中，一个 EmployeeList 的数据结构，存了工作时长和工资。两个 Department 访问，一个财务部访问工资，一个人力资源部访问工作时长，实现一个一个 IEmployee 接口。
```java
public interface IEmployee{
    void Accept(Department department)
}
class FullTimeEmployee : IEmployee{}
class PartTimeEmployee : IEmployee
```

## 空对象模式 (Null Object)
用一个空对象取代 null 对象实例的检查。

---

# 结构型
## 适配器 (Adapter)
将现有的接口适配成新的接口。解决“现存的接口”不能满足现对象的需求，就适配一下。

## 桥接 (Bridge)
抽象和实现化解耦。

例子就是遥控器和电视机 (索尼电视机，松下电视机) 。

## 组合 (Composite)
把东西看作一个整体。就是看作树一样分解。。。感觉跟递归写树的遍历一样。。

应用就是文件夹管理这样子。

## 装饰 (Decorator)
在不修改原来类的形状情况下，去扩展这个类的功能。

## 外观 (Facade)
给用户提供一个统一的接口，让用户去调用。我感觉 MVC 中的 View 就是这个。

建造者我感觉是为了防止缺胳膊断腿的情况出现，属于构造型，外观属于结构型，目的是为了提供统一的接口。说实话，实现上感觉差不多，目的和侧重点不同吧。

## 享元 (Flyweight)
用共享的方式来支持大量细粒度的对象。这些对象大部分的内部状态是相同的。

String, 线程池就是实现。String 的值就是内部状态，如果新的 String 内部状态相同，然后内存上是同一块地方。围棋中颜色是同一个 (内部状态) ，然后坐标不同 (外部状态) ，可以享元同一个内部状态。

需要一个工厂对象加以控制。

## 代理 (proxy) 
为其他对象提供一种代理来控制对这个对象的访问。

1. 适配器是为了改变所属对象的接口，代理不行。
2. 装饰器是为了增强功能，代理是为了加以控制。

---

# References
- [设计模式 —— runoob](http://www.runoob.com/design-pattern/interpreter-pattern.html)
- [Interview-Notebook/notes/设计模式.md](https://github.com/CyC2018/Interview-Notebook/blob/master/notes/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.md)