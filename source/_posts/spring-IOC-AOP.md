---
title: spring IoC AOP
date: 2017-07-04 14:48:19
tags: [spring,java]
categories: 技术
---
spring用了这么久,发现自己对这俩了解的并没有非常深刻,故填坑.
<!-- more -->
## IoC
首先来说IoC(Inversion of Control),控制反转,这名词大家很熟悉,也是Spring的核心.在没有Spring的时候,在一个对象中，如果要使用另外的对象，就必须得到它(自己new一个，或者从JNDI中查询一个)，使用完之后还要将对象销毁(比如Connection等)，对象始终会和其他的接口或类藕合起来。而IOC的思想是: Spring容器来实现这些相互依赖对象的创建、协调工作。对象只需要关系业务逻辑本身就可以了。从这方面来说，对象如何得到他的协作对象的责任被反转了(IoC).就是由spring来负责控制对象的生命周期和对象间的关系.

再来举一个简单的例子说明一下这个IoC吧:
就拿找对象来说吧,传统的找对象,得到处找自己喜欢的妹纸,然后打听电话,微信,然后想办法认识啊,找机会约会啊,然后才在一起,过程很复杂,各种耦合啊,比如约会就跟打听电话,微信,找机会约会耦合在了一起.
自从有了婚介所(Spring)我啥都不用管,只管提要求啊,约会也不用自己操心啊.只管谈恋爱就行了啊.简单明了.

整个过程不再由我自己控制，而是有婚介所这样一个类似容器的机构来控制。Spring所倡导的开发方式就是如此，所有的类都会在spring容器中登记，告诉spring你是个什么东西，你需要什么东西，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类的创建、销毁都由 spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。

最后要说一下和控制反转(IoC)关系密切的依赖注入(DI)
IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是通过DI(Dependency Injection，依赖注入)来实现的。比如对象A需要操作数据库，以前我们总是要在A中自己编写代码来获得一个Connection对象，有了 spring我们就只需要告诉spring，A中需要一个Connection，至于这个Connection怎么构造，何时构造，A不需要知道。在系统运行时，spring会在适当的时候制造一个Connection，然后像打针一样，注射到A当中，这样就完成了对各个对象之间关系的控制。A需要依赖 Connection才能正常运行，而这个Connection是由spring注入到A中的，依赖注入的名字就这么来的。那么DI是如何实现的呢？ Java 1.3之后一个重要特征是反射(reflection)，它允许程序在运行的时候动态的生成对象、执行对象的方法、改变对象的属性，spring就是通过反射来实现注入的。

## AOP
AOP(Aspect-OrientedProgramming，面向切面编程),利用“横切”的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为“Aspect”，即方面。所谓“方面”，简单地说，就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。AOP代表的是一个横向的关系，如果说“对象”是一个空心的圆柱体，其中封装的是对象的属性和行为；那么面向方面编程的方法，就仿佛一把利刃，将这些空心圆柱体剖开，以获得其内部的消息。而剖开的切面，也就是所谓的“方面”了。

实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。

# AOP相关概念
- 方面（Aspect）：一个关注点的模块化，这个关注点实现可能另外横切多个对象。事务管理是J2EE应用中一个很好的横切关注点例子。方面用Spring的Advisor或拦截器实现。
- 连接点（Joinpoint）: 程序执行过程中明确的点，如方法的调用或特定的异常被抛出；
- 通知（Advice）: 在特定的连接点，AOP框架执行的动作。许多AOP框架包括Spring都是以拦截器做通知模型，维护一个“围绕”连接点的拦截器链。Spring中定义了四个advice: BeforeAdvice, AfterAdvice, ThrowAdvice和DynamicIntroductionAdvice；
- 切入点（Pointcut）: 指定一个通知将被引发的一系列连接点的集合。AOP框架必须允许开发者指定切入点：例如，使用正则表达式。 Spring定义了Pointcut接口，用来组合MethodMatcher和ClassFilter，可以通过名字很清楚的理解，MethodMatcher是用来检查目标类的方法是否可以被应用此通知，而ClassFilter是用来检查Pointcut是否应该应用到目标类上；
- 切点标志符(designator)：execution、within、this 与 target、bean、args、@annotation；
- AOP代理对象生成:JDK动态代理和CGLib代理。CGLib所创建的动态代理对象的性能比JDK所创建的代理对象性能高不少，大概10倍，但CGLib在创建代理对象时所花费的时间却比JDK动态代理多大概8倍，所以对于singleton的代理对象或者具有实例池的代理，因为无需频繁的创建新的实例，所以比较适合CGLib动态代理技术，反之则适用于JDK动态代理技术。另外，由于CGLib采用动态创建子类的方式生成代理对象，所以不能对目标类中的final，private等方法进行处理。所以，大家需要根据实际的情况选择使用什么样的代理了。同样的，Spring的AOP编程中相关的ProxyFactory代理工厂内部就是使用JDK动态代理或CGLib动态代理的，通过动态代理，将增强（advice)应用到目标类中。


# 使用AOP
可以通过配置文件或者编程的方式来使用Spring AOP。
 
配置可以通过xml文件来进行，大概有四种方式：
1.配置ProxyFactoryBean，显式地设置advisors, advice, target等
2.配置AutoProxyCreator，这种方式下，还是如以前一样使用定义的bean，但是从容器中获得的其实已经是代理对象
3.通过<aop:config>来配置
4.通过<aop: aspectj-autoproxy>来配置，使用AspectJ的注解来标识通知及切入点
 
也可以直接使用ProxyFactory来以编程的方式使用Spring AOP，通过ProxyFactory提供的方法可以设置target对象, advisor等相关配置，最终通过 getProxy()方法来获取代理对象

# AOP使用场景
- Authentication 权限
- Caching 缓存
- Context passing 内容传递
- Error handling 错误处理
- Lazy loading　懒加载
- Debugging　　调试
- logging, tracing, profiling and monitoring　记录跟踪　优化　校准
- Performance optimization　性能优化
- Persistence　　持久化
- Resource pooling　资源池
- Synchronization　同步
- Transactions 事务