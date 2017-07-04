---
title: Spring容器启动相关
date: 2017-07-04 14:21:55
tags: [spring,java]
categories: 技术
---

前几天去面试,被问到spring容器的启动,发现是我的盲区,回来赶紧填坑.
## 手动启动IOC
```java
ClassPathResource resource = new ClassPathResource("bean.xml");  
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();  
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);  
reader.loadBeanDefinitions(resource);  
```
<!-- more --> 
第一行代码：ClassPathResource的作用是起到了资源定位的作用。通常情况下，spring的配置信息使用文件来描述，通过这样一行代码，指明了需要加载的资源的位置，并且使用Spring能够理解的Resource接口的形式将资源描述出来。
第二行代码：DefaultListableBeanFactory是一个纯粹的IoC容器类，它是这个Spring的一个基础的IoC容器类，其他的一个IoC容器都是以这个类为基础进行扩展的。这样代码只是定义了一个IoC容器，它不具有任何其他的能力。
第三行代码：XmlBeanDefinitionReader是一个配置文件读取器。它实现了BeanDefinitionReader接口，它能够按照Spring配置文件，将其中的配置信息转换为spring内部可以识别的信息（BeanDefinition）。注意，在这里它的构造函数需要一个BeanDefinitionRegistry类型的参数，BeanDefinitionRegistry接口提供了一个回调函数，通过这个回调函数可以向IoC容器注册bean的定义信息。DefaultListableBeanFactory实现了这个接口。
第四行代码：调用loadeBeanDefinitions方法，通过给定的Resource资源，从中读取出spring的配置信息，转换为BeanDefinition，然后再调用BeanDefinitionRegistry的回调函数进行注册。
通过以上的四行代码，完成了spring容器的启动。

## 容器启动过程
1. 定位
在spring中，使用统一的资源表现方式Resource。根据不同的情况进行不同的选择。上述程序中，采用了编程式的资源定位方法，使用ClassPathResource定位位于classpath下的资源文件。
2. 加载
在加载这个过程中，主要工作是读取spring配置文件，解析配置文件中的内容，将这些信息转换成为Spring内容可以理解、使用的BeanDefinition。
3. 注册
加载过配置文件后，就将BeanDefinition信息注册到BeanDefinitionRegistry中，通常情况下Spring容器的实现类都实现这个接口。

## 资源加载实现
直接上XmlBeanDefinitionReader中的loadBeanDefinitions方法的实现
```java
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {  
        Assert.notNull(encodedResource, "EncodedResource must not be null");  
        if (logger.isInfoEnabled()) {  
            logger.info("Loading XML bean definitions from " + encodedResource.getResource());  
        }  
        Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();  
        if (currentResources == null) {  
            currentResources = new HashSet<EncodedResource>(4);  
            this.resourcesCurrentlyBeingLoaded.set(currentResources);  
        }  
        if (!currentResources.add(encodedResource)) {  
            throw new BeanDefinitionStoreException(  
                    "Detected cyclic loading of " + encodedResource + " - check your import definitions!");  
        }  
        try {  
            InputStream inputStream = encodedResource.getResource().getInputStream();  
            try {  
                InputSource inputSource = new InputSource(inputStream);  
                if (encodedResource.getEncoding() != null) {  
                    inputSource.setEncoding(encodedResource.getEncoding());  
                }  
                return doLoadBeanDefinitions(inputSource, encodedResource.getResource());  
            }  
            finally {  
                inputStream.close();  
            }  
        }  
        catch (IOException ex) {  
            throw new BeanDefinitionStoreException(  
                    "IOException parsing XML document from " + encodedResource.getResource(), ex);  
        }  
        finally {  
            currentResources.remove(encodedResource);  
            if (currentResources.isEmpty()) {  
                this.resourcesCurrentlyBeingLoaded.set(null);  
            }  
        }  
    }  
```