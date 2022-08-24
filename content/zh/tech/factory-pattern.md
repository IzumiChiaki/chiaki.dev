+++
title = "工厂模式"
date = "2020-07-29T23:26:12+08:00"
tags = ["java", "design pattern"]
slug = "factory-pattern"
dropCap = false
+++
#### 重要设计模式
- 创建型模式：**单例**、**抽象工厂**、原型、建造者、**工厂方法**
- 结构型模式：适配器、桥接、**装饰**、组合、外观、享元、**代理**
- 行为型模式：模板方法、命令、访问者、迭代器、观察者、中介者、备忘录、解释器、状态、策略、职责链
##### 工厂模式
工厂顾名思义就是创建产品，根据产品是具体产品还是具体工厂可分为简单工厂模式和工厂方法模式，根据工厂的抽象程度可分为工厂方法模式和抽象工厂模式。该模式用于封装和管理对象的创建，是一种创建型模式。下面从一个具体的例子逐步深入分析，来体会三种工厂模式的应用场景和利弊。
###### 简单工厂模式
该模式对对象创建管理方式最为简单，因为其仅仅简单的对不同类对象的创建进行了一层薄薄的封装。该模式通过向工厂传递类型来指定要创建的对象，其UML类图如下：

![](https://img2020.cnblogs.com/blog/1672247/202007/1672247-20200729231858165-1307492584.png)

以手机生产为例：

**Phone接口**：手机规范标准类（AbstractProduct）
```java
public interface Phone {
    void make();
}
```
**MiPhone类**：制造小米手机（Product1）
```java
public class MiPhone implements Phone {
    public MiPhone() {
        this.make();
    }
    @Override
    public void make() {
        System.out.println("make xiaomi phone!");
    }
}
```
**IPhone类**：制造苹果手机（Product2）
```java
public class IPhone implements Phone {
    public IPhone() {
        this.make();
    }
    @Override
    public void make() {
        System.out.println("make iphone!");
    }
}
```
**PhoneFactory类**：手机代工厂（Factory）
```java
public class PhoneFactory {
    public Phone makePhone(String phoneType) {
        if(phoneType.equalsIgnoreCase("MiPhone")){
            return new MiPhone();
        }
        else if(phoneType.equalsIgnoreCase("iPhone")) {
            return new IPhone();
        }
        return null;
    }
}
```
**Demo**：
```java
public class Demo {
    public static void main(String[] arg) {
        PhoneFactory factory = new PhoneFactory();
        Phone miPhone = factory.makePhone("MiPhone");            // make xiaomi phone!
        IPhone iPhone = (IPhone)factory.makePhone("iPhone");    // make iphone!
    }
}
```
###### 工厂方法模式
与简单工厂模式中工厂负责生产所有产品相比，工厂方法模式（Factory Method）将生成具体产品的任务分发给具体的产品工厂，其UML类图如下：

![](https://img2020.cnblogs.com/blog/1672247/202007/1672247-20200729232107977-195891278.png)

也就是定义一个抽象工厂，其定义了产品的生产接口，但不负责具体的产品，将生产任务交给不同的派生类工厂。这样不用通过指定类型来创建对象了。接下来继续使用生产手机的例子来讲解该模式。其中和产品相关的Phone类、MiPhone类和IPhone类的定义不变。

**AbstractFactory接口**：生产不同产品的工厂的抽象类
```java
public interface AbstractFactory(){
    Phone makePhone();
}
```
**XiaoMiFactory类**：生产小米手机的工厂（ConcreteFactory1）
```java
public class XiaoMiFactory implements AbstractFactory{
    @Override
    public Phone makePhone() {
        return new MiPhone();
    }
}
```
**AppaleFactory类**：生产苹果手机的工厂（ConcreteFactory2）
```java
public class AppleFactory implements AbstractFactory {
    @Override
    public Phone makePhone() {
        return new IPhone();
    }
}
```
**Demo**:
```java
public class Demo {
    public static void main(String[] arg) {
        AbstractFactory miFactory = new XiaoMiFactory();
        AbstractFactory appleFactory = new AppleFactory();
        miFactory.makePhone();            // make xiaomi phone!
        appleFactory.makePhone();        // make iphone!
    }
}
```

###### 抽象工厂模式
上面两种模式不管工厂怎么拆分抽象，都只是针对一类产品Phone（AbstractProduct），如果要生成另一种产品PC，应该怎么表示呢？最简单的方式是把工厂方法中介绍的完全复制一份，不过这次生产的是PC。但同时也就意味着我们要完全复制和修改Phone生产管理的所有代码，显然这是一个笨办法，并不利于扩展和维护。抽象工厂模式（Abstract Factory）通过在AbstarctFactory中增加创建产品的接口，并在具体子工厂中实现新加产品的创建，当然前提是子工厂支持生产该产品，否则继承的这个接口可以什么也不干，其UML类图如下：

![](https://img2020.cnblogs.com/blog/1672247/202007/1672247-20200729232309569-178987843.png)

从上面类图结构中可以清楚的看到如何在工厂方法模式中通过增加新产品接口来实现产品的增加的。

接下来我们继续通过小米和苹果产品生产的例子来解释该模式。为了弄清楚上面的结构，我们使用具体的产品和工厂来表示上面的UML类图，能更加清晰的看出模式是如何演变的：

![](https://img2020.cnblogs.com/blog/1672247/202007/1672247-20200729232349595-1780801487.png)

**PC接口**：定义PC产品的接口（AbstractPC）
```java
public interface AbstractPC {
	void make();
}
```
**MiPC类**：定义小米电脑产品（MiPC）
```java
public class MiPC implements PC {
    public MiPC() {
        this.make();
    }
    @Override
    public void make() {
        System.out.println("make xiaomi PC!");
    }
}
```
**MAC类**：定义苹果电脑产品（MAC）
```java
public class MAC implements PC {
    public MAC() {
        this.make();
    }
    @Override
    public void make() {
        System.out.println("make MAC!");
    }
}
```

下面需要修改工厂相关的类定义：
**AbstractFactory接口**：增加PC产品制造接口
```java
public interface AbstractFactory(){
    Phone makePhone();
    PC makePC();
}
```
**XiaoMiFactory类**：增加小米PC的制造（ConcreteFactory1）
```java
public class XiaoMiFactory implements AbstractFactory{
    @Override
    public Phone makePhone() {
        return new MiPhone();
    }
    @Override
    public PC makePC() {
        return new MiPC();
    }
}
```
**AppleFactory类**：增加苹果PC的制造（ConcreteFactory2）
```java
public class AppleFactory implements AbstractFactory {
    @Override
    public Phone makePhone() {
        return new IPhone();
    }
    @Override
    public PC makePC() {
        return new MAC();
    }
}
```
**Demo**:
```java
public class Demo {
    public static void main(String[] arg) {
        AbstractFactory miFactory = new XiaoMiFactory();
        AbstractFactory appleFactory = new AppleFactory();
        miFactory.makePhone();            // make xiaomi phone!
        miFactory.makePC();                // make xiaomi PC!
        appleFactory.makePhone();        // make iphone!
        appleFactory.makePC();            // make MAC!
    }
}
```

##### 总结
+ 工厂模式的意义：将实例化对象的代码提取出来，放到一个类中统一管理和维护，达到主项目的依赖关系解耦，从而提高项目的扩展和维护性。
+ 三种工厂模式：**简单工厂模式**、**工厂方法模式**、**抽象工厂模式**
+ 设计模式的**依赖抽象**原则