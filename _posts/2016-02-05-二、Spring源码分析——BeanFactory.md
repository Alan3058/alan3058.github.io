---
layout: post
title: [二、Spring源码分析——BeanFactory]
categories: [Spring]
tags: [spring,源码分析,beanfactory]
fullview: false
---
# 1、BeanFactory类图

BeanFactory类图如下

![](http://file.ctosb.com/upload/image/20170705/1499240063366039182.png)

从上图可以看出BeanFactory主要实现类是XmlBeanFactory（Spring3.1建议弃用，可以使用DefaultListableBeanFactory和XmlBeanDefinitionReader编程实现）和DefaultListableBeanFactory。

# 2、BeanFactory编程式实现

基本IOC容器BeanFactory的编程式实现方式大致可分为三个步骤

1.加载xml资源 Resource定位xml

2.BeanDefinition读取和加载 BeanDefinitionParserDelegate代理对象去读取

3.BeanDefinition注册 BeanDefinitionRegistry去注册

创建bean.xml，内容如下
<?xml version="1.0" encoding="UTF-8"?> <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.springframework.org/schema/beans" xmlns:aop="http://www.springframework.org/schema/aop" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd"> <bean id="person" class="com.test.bean.Person"> <property name="name" value="ylxy"/> <property name="age" value="25"/> </bean> </beans>

创建Person类，代码如下
package com.test.bean; public class Person { private String name; private int age; public String getName() { return name; } public void setName(String name) { this.name = name; } public int getAge() { return age; } public void setAge(int age) { this.age = age; } public void info(){ System.out.println("name:"+getName()+" age:"+getAge()); } }

创建junit测试代码，内容如下
@Test public void testBeanFactory(){ ClassPathResource resource = new ClassPathResource("bean.xml"); DefaultListableBeanFactory bf = new DefaultListableBeanFactory(); XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(bf); reader.loadBeanDefinitions(resource); Person p = bf.getBean("person", Person.class); p.info(); }

最终测试结果如下

![](http://file.ctosb.com/upload/image/20170705/1499240077889001017.png)

# 3、IOC配置文件处理（BeanDefinition解析与注册）

从编程代码上看，可分以下几个步骤

1.Resource定位（即Resource包装spring xml配置文件）。

2.BeanDefinition解析和注册（由XmlBeanDefinitionReader类处理解析和注册）。

3.获取Bean（由BeanFactory获取，依赖注入）。

从上面可知BeanDefinition的解析和注册入口在XmlBeanDefinitionReader的loadBeanDefinitions(Resource resource)方法上。

1.在loadBeanDefinitions方法中，调用该类的doLoadBeanDefinitions(InputSource inputSource, Resource resource)方法。

2.在doLoadBeanDefinitions方法中通过DocumentLoader（实际是DefaultDocumentLoader对象）加载xml文件成Document类，之后调用该类的registerBeanDefinitions(Document doc, Resource resource)方法去解析加载和注册BeanDefinition。

3.在registerBeanDefinitions方法中创建DefaultBeanDefinitionDocumentReader对象，并调用registerBeanDefinitions( Document doc, XmlReaderContext readerContext)去解析注册BeanDefinition，解析时会对不同的xml元素进行不同的解析，比如<import/><alias/><bean/><beans/>等元素。

4.最终实际解析BeanDefinition的实现是创建BeanDefinitionParserDelegate委托对象，调用方法parseBeanDefinitionElement(Element ele)去解析BeanDefinition。

5.注册对象是在BeanDefinitionReaderUtils.registerBeanDefinition( BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)中实现。

BeanDefinition解析实现：调用BeanDefinitionParserDelegate.parseBeanDefinitionElement(Element ele)去解析xml成BeanDefinition，并将BeanDefinition封装成BeanDefinitionHolder。

BeanDefinition注册实现：调用BeanDefinitionRegistry.registerBeanDefinition(String beanName, BeanDefinition beanDefinition)，由于DefaultListableBeanFactory实现了该接口的方法，故而实现注册BeanDefinition是在DefaultListableBeanFactory中。在该类中通过将BeanDefinition存储到一个HashMap集合中，完成对BeanDefinition的注册。

# 4、依赖注入

依赖注入和控制反转是同一个概念。当某个A javaBean需要B javaBean的引用去协助处理一些事情的时候，在传统程序设计中，是需要A去new一个B对象。然而在spring中却不一样，创建B对象的工作不是由A来new，而是由spring IOC容器去创建，然后将创建的B对象注入到A中。

spring的依赖注入入口也就是BeanFactory的getBean（）方法。getBean（）方法最后调用doGetBean（）方法，即doGetBean（）方法是依赖注入的真实入口，代码如下。
protected <T> T doGetBean( final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly) throws BeansException { final String beanName = transformedBeanName(name); Object bean; // Eagerly check singleton cache for manually registered singletons. Object sharedInstance = getSingleton(beanName); if (sharedInstance != null && args == null) { if (logger.isDebugEnabled()) { if (isSingletonCurrentlyInCreation(beanName)) { logger.debug("Returning eagerly cached instance of singleton bean '" + beanName + "' that is not fully initialized yet - a consequence of a circular reference"); } else { logger.debug("Returning cached instance of singleton bean '" + beanName + "'"); } } bean = getObjectForBeanInstance(sharedInstance, name, beanName, null); } else { // Fail if we're already creating this bean instance: // We're assumably within a circular reference. if (isPrototypeCurrentlyInCreation(beanName)) { throw new BeanCurrentlyInCreationException(beanName); } // Check if bean definition exists in this factory. BeanFactory parentBeanFactory = getParentBeanFactory(); //检查这个bean是否在当前这个BeanFactory中，如果当前BeanFactory的不包含这个bean，则去父级BeanFactory查找。 if (parentBeanFactory != null && !containsBeanDefinition(beanName)) { // Not found -> check parent. String nameToLookup = originalBeanName(name); if (args != null) { // Delegation to parent with explicit args. return (T) parentBeanFactory.getBean(nameToLookup, args); } else { // No args -> delegate to standard getBean method. return parentBeanFactory.getBean(nameToLookup, requiredType); } } if (!typeCheckOnly) { markBeanAsCreated(beanName); } try { final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName); checkMergedBeanDefinition(mbd, beanName, args); // Guarantee initialization of beans that the current bean depends on. String[] dependsOn = mbd.getDependsOn(); //查找这个BeanDefinition是否有依赖对象，如果有依赖的对象，则IOC容器先创建依赖对象。 if (dependsOn != null) { for (String dependsOnBean : dependsOn) { getBean(dependsOnBean); registerDependentBean(dependsOnBean, beanName); } } // Create bean instance. if (mbd.isSingleton()) { //获取单例对象 sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() { public Object getObject() throws BeansException { try { //创建Bean的入口方法 return createBean(beanName, mbd, args); } catch (BeansException ex) { // Explicitly remove instance from singleton cache: It might have been put there // eagerly by the creation process, to allow for circular reference resolution. // Also remove any beans that received a temporary reference to the bean. destroySingleton(beanName); throw ex; } } }); //这里创建动态代理对象 bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd); } else if (mbd.isPrototype()) { // It's a prototype -> create a new instance. Object prototypeInstance = null; try { beforePrototypeCreation(beanName); prototypeInstance = createBean(beanName, mbd, args); } finally { afterPrototypeCreation(beanName); } bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd); } else { String scopeName = mbd.getScope(); final Scope scope = this.scopes.get(scopeName); if (scope == null) { throw new IllegalStateException("No Scope registered for scope '" + scopeName + "'"); } try { Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() { public Object getObject() throws BeansException { beforePrototypeCreation(beanName); try { return createBean(beanName, mbd, args); } finally { afterPrototypeCreation(beanName); } } }); bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd); } catch (IllegalStateException ex) { throw new BeanCreationException(beanName, "Scope '" + scopeName + "' is not active for the current thread; " + "consider defining a scoped proxy for this bean if you intend to refer to it from a singleton", ex); } } } catch (BeansException ex) { cleanupAfterBeanCreationFailure(beanName); throw ex; } } // Check if required type matches the type of the actual bean instance. if (requiredType != null && bean != null && !requiredType.isAssignableFrom(bean.getClass())) { try { return getTypeConverter().convertIfNecessary(bean, requiredType); } catch (TypeMismatchException ex) { if (logger.isDebugEnabled()) { logger.debug("Failed to convert bean '" + name + "' to required type [" + ClassUtils.getQualifiedName(requiredType) + "]", ex); } throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass()); } } return (T) bean; }

从上述代码可以看出创建对象是由DefaultSingletonBeanRegistry的getSingleton方法去创建，然后回调ObjectFactory的getObject方法，ObjectFactory的getObject方法又调用了createBean方法。也就是说实际创建对象的方法是AbstractAutowireCapableBeanFactory类的createBean方法，该方法内又调用doCreateBean去创建Bean。

1.调用createBeanInstance方法去创建BeanWrap（BeanWrap将Bean实例包装了），该方法通过Java反射机制调用Bean的构造函数，完成Bean对象的创建，并将Bean包装到一个BeanWrap对象中。

2.调用populateBean方法，给bean的属性初始化，也就是<bean/>标签中配置的属性值。

3.调用initializeBean方法，初始化Bean。

3.1.invokeAwareMethods方法，如果Bean实现了Aware，则调用对应的Aware接口方法。

3.2.applyBeanPostProcessorsBeforeInitialization方法，如果BeanFactory配置了BeanPostProcessor，则执行BeanPostProcessor的postProcessBeforeInitialization方法。

3.3.invokeInitMethods方法，如果Bean实现了InitializingBean接口，则调用执行InitializingBean的接口方法。

3.4.applyBeanPostProcessorsAfterInitialization方法，如果BeanFactory配置了BeanPostProcessor，则执行BeanPostProcessor的postProcessAfterInitialization方法。

4.registerDisposableBeanIfNecessary方法，如果Bean实现了DisposableBean接口或者Bean需要destroy方法，则给Bean注册DisposableBean功能（即把bean名称和销毁方法保存在一个Map集合中）。

5.getObjectForBeanInstance(sharedInstance, name, beanName, mbd)方法，创建bean完成后，如果创建的bean实例是一个FactoryBean（相当于工厂模式的Factory，用来生产Bean实例），则会再调用FactoryBean的getObject()方法来生成实际需要的bean实例。也就是说如果getBean("factoryBean")会首先获取FactoryBean实例，并且在通过FactoryBean工厂的getObject方法去获取Bean，如果是getBean（“&factoryBean”）,则spring会直接返回一个FactoryBean实例。

# 5、总结

对于BeanDefinition的解析和注册过程，首先将xml配置文件加载进Resource中，然后将Resource解析成Document对象，最后通过BeanDefinitionParserDelegate委托对象去解析Document对象，并将解析的BeanDefinition包装成BeanDefinitionHolder。最后通过BeanDefinitionRegistry接口将BeanDefinition存储到HashMap集合中，以完成BeanDefinition的的注册，具体实现在DefaultListableBeanFactory类中，该类实现了BeanDefinitionRegistry接口。

对于依赖注入，spring主要是使用Java的反射机制来创建bean对象，并为bean进行一些初始化操作。

源码见如下附件

![](http://ctosb.com/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif)[cygoattest.zip](http://file.ctosb.com/upload/file/20170705/1499240213131066792.zip "cygoattest.zip")
