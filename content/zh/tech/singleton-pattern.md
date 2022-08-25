+++
title = "单例模式"
date = "2020-07-30T23:31:29+08:00"
tags = ["java", "design pattern"]
slug = "singleton-pattern"
dropCap = false
+++

Java的单例模式是一种常见设计模式，单例模式的写法主要有：**懒汉式单例**、**饿汉式单例**、**登记式单例**。单例模式有以下特点：

- 单例类只能有一个实例
- 单例类必须自己创建自己的唯一实例
- 单例类必须给所有其他对提供这一实例

单例模式确保某个类只有一个实例，而且自行实例化并向整个系统提供这个实例。在计算机系统中，线程池、缓存、日志对象、对话框、打印机、显卡的驱动程序对象常被设计成单例。这些应用都或多或少具有资源管理器的功能。每台计算机可以有若干个打印机，但只能有一个Printer
Spooler，以避免两个打印作业同时输出到打印机中。每台计算机可以有若干通信端口，系统应当集中管理这些通信端口，以避免一个通信端口同时被两个请求同时调用。总之，选择单例模式就是为了避免不一致状态。

##### 饿汉式

**Singleton01类（静态常量）**：

```java
public class Singleton01 {
    // 将构造器私有化，防止直接new
    private Singleton01() {
    }

    // 创建一个静态Singleton01对象实例
    private static Singleton01 instance = new Singleton01();

    // 提供一个public的静态方法，可以返回instance
    public static Singleton01 getInstance() {
        return instance;
    }
}
```

**Singleton02类（静态代码块）**：

```java
public class Singleton02 {
    private Singleton02() {
    }

    private static Singleton instance;

    // 使用静态代码块
    static {
        instance = new Singleton();
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

- 优点：写法比较简单，在类装载的时候就完成实例化，避免了线程同步问题。
- 缺点：在类装载的时候就完成实例化，没有达到 Lazy Loading 的效果。如果从始至终从未使用过这个实例，则会造成内存的浪费。

结论：单例模式**可用**，但**可能造成内存浪费**。

##### 懒汉式

**Singleton03类（线程不安全）**：

```java
public class Singleton03 {
    private Singleton03() {
    }

    private static Singleton03 singleton;

    // 当调用getInstance才创建单例对象
    public static Singleton03 getInstance() {
        if (singleton == null) {
            singleton = new Singleton03();
        }
        return singleton;
    }
}
```

- 优点：起到了Lazying Loading的效果。
- 缺点：只能在单线程下使用。如果在多线程下，一个线程进入了`if (sigleton == null)`
  判断语句块还未来得及往下执行，另一个线程也通过了这个判断语句，这时就会产生多个实例。所以在多线程环境下不可使用这种方式。

结论：实际开发中**不要**使用这种方式。

**Singleton04类（同步方法）**：

```java
public class Singleton04 {
    private Singleton04() {
    }

    private static Singleton04 singleton;

    // 使用synchronized给方法加锁
    public static synchronized Singleton04 getInstance() {
        if (singleton == null) {
            singleton = new Singleton04();
        }
        return singleton;
    }
}
```

- 优点：**解决了线程不安全问题**。
- 缺点：效率太低。每个线程在想获得类的实例时，执行`getInstance()`
  方法都要进行同步，其实该方法只执行一次实例化代码就够了，后面的线程想过的该类实例，直接`return`就行，方法进行同步导致效率太低。

结论：实际开发中**不推荐**使用这种方式。

**Singleton05类（同步代码块）**：

```java
public class Singleton05 {
    private Singleton05() {
    }

    private static Singleton05 singleton;

    public static Singleton05 getInstance() {
        if (singleton == null) {
            synchronized (Singleton05.class) {
                singleton = new Singleton05();
            }
        }
        return singleton;
    }
}
```

- 优点：本意是对第四种实现方式的改进。
- 缺点：并**不能起到线程同步的作用**。与第三种实现方式一致，如果在多线程下，一个线程进入了`if (sigleton == null)`
  判断语句块还未来得及往下执行，另一个线程也通过了这个判断语句，这时就会产生多个实例。

结论：实际开发中**不能**使用这种方式。

##### 双检锁

**Singleton06类（双重检查）**：

```java
public class Singleton06 {
    private Singleton06() {
    }

    private volatile static Singleton06 singleton;

    public static Singleton06 getInstance() {
        // 第一重检查
        if (singleton == null) {
            synchronized (Singletion06.class) {
                // 第二重检查
                if (singleton == null) {
                    singleton = new Singleton06();
                }
            }
        }
        return singleton;
    }
}
```

- 保证线程安全。Double-Check是多线程开发中常使用到的，如代码中所示，进行了两次`if(singleton == null)`检查，这样就可以保证线程安全。
- 实例化代码只用执行一次，后面有线程再次访问时，判断`if(singleton == null)`直接`return`实例化对象，避免反复进行方法同步。
- 一定要**使用`volatile`关键字防止指令重排**
- 线程安全：延迟加载，效率较高。

结论：实际开发中**推荐**使用这种单例设计模式。

##### 静态内部类

**Singleton07类（静态内部类）**：

```java
public class Singleton07 {
    private Singleton07() {
    }

    private static class SingletonInstance {
        private static final Singleton07 INSTANCE = new Singleton07();
    }

    public static Singleton07 getInstance() {
        return SingletonInstance.INSTANCE;
    }
}
```

- 采用类装载的机制来保证初始化实例时只有一个线程。
- 静态内部类方式在`Singleton07`类被装载时并不会立即实例化，而是在需要实例化时调用`getInstance`
  方法，才会装载`SingletonInstance`类，从而完成`Singleton07`的实例化。
- 类的静态属性只会在第一次加载类的时候初始化，所以这里JVM帮助保证了**线程的安全性**，在类进行初始化时，别的线程无法进入。
- 优点：**避免了线程不安全，利用静态内部类特点实现延迟加载，效率高**。

结论：实际开发中**推荐使用**。

##### 枚举

**Singleton08类（枚举）**：

```java
enum Singleton08 {
    INSTANCE;

    public void anyMethod() {
    }
}
```

- 借助JDK1.5中添加的枚举来实现单例模式。**不仅能够避免多线程同步问题，而且还能防止反序列化重新创建新的对象**。
- 《Effective Java》的作者Josh Blosh提倡的方式。

结论：实际开发中**推荐使用**。

##### 单例模式的JDK应用

JDK中的`java.lang.Runtime`是典型的单例模式（饿汉式）

![singleton-runtime.png](/images/singleton-runtime.png)

##### 总结

- 饿汉式没有线程同步问题，可以使用，但是会有内存浪费问题。
- 懒汉式（同步方法）线程安全但效率低，而懒汉式（同步代码块）线程不安全，因此懒汉式都比较少用。
- 双检锁线程安全，延迟加载，效率高。
- 若明确需要Lazying Loading，可以考虑静态内部类，其线程安全，效率高。
- 枚举方式线程安全，当涉及到要防止反序列化重新创建对象时，建议使用。