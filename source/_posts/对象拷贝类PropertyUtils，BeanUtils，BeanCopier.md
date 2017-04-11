---
title: 对象拷贝类PropertyUtils，BeanUtils，BeanCopier
date: 2017-03-27 15:42:15
tags: [java,反射]
categories: 技术
---
### 目前流行的较为公用认可的工具类：

#### Apache的两个版本：（反射机制）
```java
org.apache.commons.beanutils.PropertyUtils.copyProperties(Object dest, Object orig)
org.apache.commons.beanutils.BeanUtils.copyProperties(Object dest, Object orig)
```
<!-- more --> 
#### Spring版本：（反射机制）
```java
org.springframework.beans.BeanUtils.copyProperties(Object source, Object target, Class editable, String[] ignoreProperties)
```
#### cglib版本：（使用动态代理，效率高）
```java
net.sf.cglib.beans.BeanCopier.copy(Object paramObject1, Object paramObject2, Converter paramConverter)
```
### 原理简介

#### 反射类型：（apache）

都使用静态类调用，最终转化虚拟机中两个单例的工具对象。
```java
public BeanUtilsBean()
{
  this(new ConvertUtilsBean(), new PropertyUtilsBean());
}
```
ConvertUtilsBean可以通过ConvertUtils全局自定义注册。
```java
ConvertUtils.register(new DateConvert(), java.util.Date.class);
```
PropertyUtilsBean的copyProperties方法实现了拷贝的算法。
1、  动态bean：
```java
orig instanceof DynaBean：Object value = ((DynaBean)orig).get(name);
```
然后把value复制到动态bean类
2、  Map类型：orig instanceof Map：key值逐个拷贝
3、  其他普通类：：从beanInfo【每一个对象都有一个缓存的bean信息，包含属性字段等】取出name，然后把sourceClass和targetClass逐个拷贝
#### Cglib类型：BeanCopier
```java
copier = BeanCopier.create(source.getClass(), target.getClass(), false);
copier.copy(source, target, null);
```
Create对象过程：产生sourceClass-》TargetClass的拷贝代理类，放入jvm中，所以创建的代理类的时候比较耗时。最好保证这个对象的单例模式，可以参照最后一部分的优化方案。
创建过程：源代码见
```java
jdk：net.sf.cglib.beans.BeanCopier.Generator.generateClass(ClassVisitor)
```
1、  获取sourceClass的所有public get 方法-》PropertyDescriptor[] getters
2、  获取TargetClass 的所有 public set 方法-》PropertyDescriptor[] setters
3、  遍历setters的每一个属性，执行4和5
4、  按setters的name生成sourceClass的所有setter方法-》PropertyDescriptor getter【不符合javabean规范的类将会可能出现空指针异常】
5、  PropertyDescriptor[] setters-》PropertyDescriptor setter
6、  将setter和getter名字和类型 配对，生成代理类的拷贝方法。
Copy属性过程：调用生成的代理类，代理类的代码和手工操作的代码很类似，效率非常高。
