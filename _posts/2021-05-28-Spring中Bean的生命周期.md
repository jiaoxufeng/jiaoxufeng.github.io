---
layout:     post
title:      Spring中Bean的生命周期
date:       2021-5-28
author:     焦旭峰
header-img: img/the-first.png
catalog:   true
tags:
    - Spring
---



> # Spring中Bean的生命周期

![img](https://pic4.zhimg.com/80/754a34e03cfaa40008de8e2b9c1b815c_720w.jpg?source=1940ef5c)

1、Spring对Bean进行实例化；

2、Spring将值和Bean的引用注入到Bean对应的属性中；

3、如果bean实现了BeanNameAware接口，Spring将bean的ID传递给setBean-Name()方法；

**（实现BeanNameAware清主要是为了通过Bean的引用来获得Bean的ID，一般业务中是很少有用到Bean的ID的**）

4、如果bean实现了BeanFactoryAware接口，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入；

**（实现BeanFactoryAware 主要目的是为了获取Spring容器，如Bean通过Spring容器发布事件等）**

5、如果bean实现了ApplicationContextAware接口，Spring将调用setApplicationContext()方法，将bean所在的应用上下文的引用传入进来；

**(作用与BeanFactory类似都是为了获取Spring容器，不同的是Spring容器在调用setApplicationContext方法时会把它自己作为setApplicationContext 的参数传入，而Spring容器在调用setBeanDactory前需要程序员自己指定（注入）setBeanDactory里的参数BeanFactory )**

6、如果bean实现了BeanPostProcessor接口，Spring将调用它们的post-ProcessBeforeInitialization()方法；

**（作用是在Bean实例创建成功后对进行增强处理，如对Bean进行修改，增加某个功能）**

7、如果bean实现了InitializingBean接口，Spring将调用它们的after-PropertiesSet()方法。类似地，如果bean使用initmethod声明了初始化方法，该方法也会被调用；

8、如果bean实现了BeanPostProcessor接口，Spring将调用它们的post-ProcessAfterInitialization()方法；

**（作用与6的一样，只不过6是在Bean初始化前执行的，而这个是在Bean初始化后执行的，时机不同 )**

9、此时，bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁；

10、如果bean实现了DisposableBean接口，Spring将调用它的destroy()接口方法。同样，如果bean使用destroy-method声明了销毁方法，该方法也会被调用。
