---
title: java内存溢出总结
date: 2017-03-27 17:06:24
tags: java
categories: 技术
---
首先我们来说说java虚拟机的构成吧:

他是由程序计数器,堆,java虚拟机栈,本地方法栈,方法区这几块构成的,
<!-- more --> 
- 堆是用来存放对象和数组的,这里我们就可以设计一个无限创造对象来模拟堆溢出.

```java
/**

 * 堆溢出 

 * Exception in thread "main" java.lang.OutOfMemoryError: Java heap space

 * @author chenwulou

 */

public class HeapLeak {

    public static void main(String[] args) {
        ArrayList list = new ArrayList();
        while (true) {
            list.add(new HeapLeak.method());
        }
    }
    
    static class method {
    
    }
}
```


- 栈是用来存放基本数据类型，对象引用，方法等等这些东西的,这里我们就能用无限递归来模拟栈溢出

```java
/**

 * 栈溢出

 * Exception in thread "main" java.lang.StackOverflowError

 * @author chenwulou

 */

public class StackLeak {

 public static void main(String[] args) {

 method();

 }

 public static void method() {

 method();

 }

}

```

- 方法区用来存放每个class的结构,比如说运行时常量池、域、方法数据、方法体、构造函数、包括类中的专用方法、实例初始化、接口初始化。cglib可以直接操作字节码，也可以动态产生Class，下面通过cglib来演示。


```java
import java.lang.reflect.Method;

import net.sf.cglib.proxy.Enhancer;

import net.sf.cglib.proxy.MethodInterceptor;

import net.sf.cglib.proxy.MethodProxy;


/**

 * 方法区溢出

 * Exception in thread "main" java.lang.NoClassDefFoundError

 * @author chenwulou

 */

public class MethodAreaLeak {

public static void main(String[] args) {

while (true) {

    Enhancer enhancer = new Enhancer();
    
    enhancer.setSuperclass(OOMObject.class);
    
    enhancer.setUseCache(false);
    
    enhancer.setCallback(new MethodInterceptor() {
    
        public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy)
        
        throws Throwable {
        
            return proxy.invokeSuper(obj, args);
        
        }
    
    });
    
    enhancer.create();

}

}

class OOMObject {

}

}
```