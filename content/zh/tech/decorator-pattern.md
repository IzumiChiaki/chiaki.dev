+++
title = "装饰者模式"
date = "2020-08-01T23:12:29+08:00"
tags = ["java", "design pattern"]
slug = "decorator-pattern"
dropCap = false
+++

##### 装饰者模式定义

**动态地将新功能附加到对象上**。在对象功能扩展方面，他比继承更有弹性，装饰者模式也体现了**开闭原则**（OCP），其UML类图如下所示：

![decorator-1.png](/images/decorator-1.png)

##### 案例：☕订单项目

​ 1）咖啡种类/单品咖啡：Espresso、Decaf、DarkRoast、HouseBlend

​ 2）调料：Milk、Soy、Mocha、Whip

​ 3）要求在扩展新的咖啡种类是，具有良好的扩展性、改动方便、维护方便

​ 4）需要计算咖啡订单的费用：客户可以点**单品咖啡**，也可以**单品咖啡+调料组合**

利用装饰者设计模式来实现☕订单项目，其UML类图如下：

![decorator-2.png](/images/decorator-2.png)

##### 装饰者模式下的订单

不管是什么形式的单品咖啡+调料组合都可以通过**递归**进行方便的组合和维护。

![decorator-3.png](/images/decorator-3.png)

##### 装饰者模式的JDK应用

**Java的IO结构**

![decorator-4.png](/images/decorator-4.png)

##### 装饰者模式的设计原则

- 多用组合，少用继承。
- 对扩展开放，对修改关闭（OCP）。