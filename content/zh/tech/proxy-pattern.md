+++
title = "代理模式"
date = "2020-07-31T23:31:29+08:00"
tags = ["java", "design pattern"]
slug = "proxy-pattern"
dropCap = false
+++

##### 代理模式基本介绍

+ 代理模式（Proxy）：为一个对象**提供一个替身**，以控制对这个对象的访问，即通过代理对象访问目标对象。这样做的好处在于：**
  可以在目标对象实现的基础上，增强额外的功能操作，即扩展目标对象的功能**。
+ 被代理的对象可以是**远程对象**、**创建开销大的对象**或者**需要安全控制的对象**。
+ 代理模式有三种不同形式，主要有**静态代理**、**动态代理**（JDK代理/接口代理）以及**Cglib代理**
  （可以在内存中动态地创建对象，而不需要实现接口，也属于动态代理的范畴）。

![proxy.png](/images/proxy.png)

##### 静态代理

静态代理在使用时，需要定义接口或者父类，被代理对象（即目标对象）与代理对象一起实现相同的接口或者继承相同的父类。

##### 案例：👩‍🏫授课（一）

![static-proxy.png](/images/static-proxy.png)

**ITeacherDao接口**：

```java
public interface ITeacherDAO {
    void teach();
}
```

**目标对象TeacherDao实现ITeacherDao接口**：

```java
public class TeacherDao implements ITeacherDao {

    @Override
    public void teach() {
        System.out.println("老师授课中...");
    }
}
```

**使用静态代理，创建代理对象TeacherDaoProxy同样实现ITeacherDao接口**：

```java
public class TeacherDaoProxy implements ITeacherDao {
    // 目标对象
    private ITeacherDao target;

    // 构造方法
    public TeacherDaoProxy(ITeacherDao target) {
        this.target = target;
    }

    // 实现接口重写teach方法
    @Override
    public void teach() {
        System.out.println("开始代理并完成一些操作...");
        target.teach();
        System.out.println("提交...");
    }

}
```

**客户端Client**：

```java
public class Client {
    public static void main(String[] args) {
        // 创建目标对象
        TeacherDao teacherDao = new TeacherDao();

        // 创建代理对象, 同时将被代理对象传递给代理对象
        TeacherDaoProxy teacherDaoProxy = new TeacherDaoProxy(teacherDao);
        // 通过代理对象调用目标对象的方法
        // 即执行的是代理对象的方法，代理对象再去调用目标对象的方法 
        teacherDaoProxy.teach();
    }
}
```

- 优点：在不修改目标对象的功能前提下，能通过代理对象对目标对象功能扩展。
- 缺点：因为代理对象需要与目标对象实现一样的接口，所以会有很多代理类。一旦接口增加方法，目标对象与代理对象都要维护。

##### 动态代理

+ 代理对象不需要实现接口，但是目标对象要实现接口，否则不能用动态代理。
+ 代理对象的生成，是利用JDK的API，动态地在内存中构建代理对象。
+ 动态代理也叫做JDK代理，或接口代理。

**JDK中生成代理对象的API**：

代理类所在的包`java.lang.reflect.Proxy`。JDK实现代理只需使用`newProxyInstance`方法，该方法需要接收三个参数：

```
static Object newProxyInstance(ClassLoader loader, Class<?>[] interface, InvocationHandler h)
```

> - `ClassLoader loader` ： 指定当前目标对象使用的类加载器, 获取加载器的方法固定
> - `Class<?>[] interfaces`: 目标对象实现的接口类型，使用泛型方法确认类型
> - `InvocationHandler h` : 事情处理，执行目标对象的方法时会触发事情处理器方法，把当前执行的目标对象方法作为参数传入

##### 案例：👩‍🏫授课（二）

将👩‍🏫授课案例（一）改进成为动态代理。

![dynamic-proxy.png](/images/dynamic-proxy.png)

**ITeacherDao接口**：

```java
public interface ITeacherDAO {
    void teach();

    void sayHello(String name);
}
```

**目标对象TeacherDao实现ITeacherDao接口**：

```java
public class TeacherDao implements ITeacherDao {

    @Override
    public void teach() {
        System.out.println("老师授课中...");
    }

    @Override
    public void sayHello(String name) {
        System.out.println("hello" + name);
    }
}
```

**使用动态代理，创建代理对象ProxyFactory，调用newProxyInstance方法**：

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyFactory {
    // 维护一个目标对象
    private Object target;

    // 构造方法
    public TeacherDaoProxy(Object target) {
        this.target = target;
    }

    // 给目标对象生成代理对象
    public Object getProxyInstance() {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                (proxy, method, args) -> {
                    System.out.println("JDK代理开始...");
                    // 反射机制调用目标对象的方法
                    Object returnVal = method.invoke(target, args);
                    System.out.println("JDK代理提交...");
                    return returnVal;
                });
    }
}
```

**客户端Client**：

```java
public class Client {
    public static void main(String[] args) {
        // 创建目标对象
        TeacherDao teacherDao = new TeacherDao();

        // 创建代理对象并将Object强制转型为ITeacherDao
        ITeacherDao proxyInstance = (ITeacherDao) new ProxyFactory(target).getProxyInstance();

        // 通过代理对象调用目标对象的方法
        proxyInstance.sayHello("Tom");
    }
}
```

- 优点：代理对象无需实现接口。
- 缺点：如果目标对象未实现任何接口，则不能采用通过动态代理的。

##### Cglib代理

**Cglib基本介绍：**

静态代理和JDK代理模式都要求目标对象是实现一个接口，但是有时候目标对象只是一个单独的对象并没有实现任何的接口，这个时候可**
使用目标对象子类来实现代理**，这就是Cglib代理。

- Cglib代理也叫作**子类代理**它是在内存中构建一个子类对象从而实现对目标对象功能扩展，有些书中将Cglib代理归属到动态代理 。
- Cglib是一个强大的高性能的代码生成包，它可以在运行期扩展 Java 类与实现 Java接口。它广泛的被许多AOP 的框架使用，例如 Spring
  AOP实现方法拦截。
- 在AOP编程中选择代理模式的原则：目标对象需要实现接口用JDK代理，不需要实现接口用Cglib代理。
- Cglib底层是通过使用字节码处理框架ASM转换字节码并生成新的类。

**Cglib代理模式实现步骤：**

- Cglib的相关依赖：`asm.jar`，`asm-commons.jar`，`asm-tree.jar`，`cglib-3.3.0.jar`。
- 在内存中构建子类，注意代理类不能为final。
- 目标对象的方法如果为final/static，那么就不会被拦截，即不会执行目标对象额外的业务方法。

##### 案例：👩‍🏫授课（三）

将前面的案例用Cglib代理实现。

![cglib-proxy.png](/images/cglib-proxy.png)

**目标对象TeacherDao，无需实现接口：**

```java
public class TeacherDao {
    public String teach() {
        System.out.println("老师授课中...Cglib代理无需实现接口...");
        return "Hello";
    }
}
```

**创建代理对象ProxyFactory，使用Cglib代理：**

```java
import java.lang.reflect.Method;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class ProxyFactory implements MethodInterceptor {
    // 创建一个目标对象
    private Object target;

    // 构造方法
    public ProxyFactory(Object target) {
        this.target = target;
    }

    // 返回一个target对象的代理对象
    public Object getProxyInstance() {
        // 1. 创建一个工具类
        Enhancer enhancer = new Enhancer();
        // 2. 设置父类
        enhancer.setSuperclass(target.getClass());
        // 3. 设置回调函数
        enhancer.setCallback(this);
        // 4. 创建子类对象，即代理对象
        return enhancer.create();
    }

    // 重写intercept方法 会调用目标对象的方法
    @Override
    public Object intercept(Object obj, Method method, Object[] objs, MethodProxy methodProxy) throws Throwable {
        System.out.println("Cglib代理模式...开始...");
        Object returnVal = method.invoke(target, args);
        System.out.println("Cglib代理模式...提交...");
        return returnVal;
    }
}
```

**客户端Client：**

```java
public class Client {
    public static void main(String[] args) {
        // 创建目标对象
        TeacherDao target = new TeacherDao();
        // 获取到代理对象并将目标对象作为参数传递给代理对象
        TeacherDao proxyInstance = (TeacherDao) new ProxyFactory(target).getProxyInstance();
        //执行代理对象的方法并触发intercept方法从而实现对目标对象方法的调用
        String res = proxyInstance.teach();
        System.out.println("res = " + res);
    }
}
```

Cglib代理相比于静态代理和JDK代理，其对目标对象实现是否实现接口没有要求。

##### 代理模式变体

几种常见的Proxy变体：

- 防火墙代理：内网通过代理穿透防火墙实现对公网的访问
- 缓存代理：比如当请求图片文件等资源时，先到缓存代理取，如果取到则ok，如果娶不到，再到公网或者数据库取，然后缓存。
- 远程代理：远程对象的本地代表，通过它可以把远程对象方本地对象来调用。远程代理通过网络和真正的远程对象沟通信息。
- 同步代理：主要使用在多线程编程中，完成多线程间同步工作。