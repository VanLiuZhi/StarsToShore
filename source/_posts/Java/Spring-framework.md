---
title: Spring framework
date: 2019-11-01 00:00:00
author: vanliuzh
top: true
cover: false
toc: true
mathjax: false
summary: IoC也称为依赖注⼊(dependency injection, DI)。它是⼀个对象定义依赖关系的过程，也就是说，对象只通过构造函数参数。⼯⼚⽅法的参数或对象实例构造或从⼯⼚⽅法返回后在对象实例上设置的属性来定义它们所使⽤的其他对象
categories: Java
tags: [Java, Note]
reprintPolicy: cc_by
---

Spring framework

<!-- more -->

## Ioc 设计理念

IoC也称为依赖注⼊(dependency injection, DI)。它是⼀个对象定义依赖关系的过程，也就是说，对象只通过构造函数参数、⼯⼚⽅法的参数或对象实例构造或从⼯⼚⽅法返回后在对象实例上设置的属性来定义它们所使⽤的其他对象。

然后容器在创建bean时注⼊这些依赖项。这个过程基本上是bean的逆过程，因此称为控制反转(IoC) 在Spring中，构成应⽤程序主⼲并由Spring IoC容器管理的对象称为bean。

bean是由Spring IoC容器实例化、组装和管理的对象。IoC容器设计理念: 通过容器统⼀对象的构建⽅式，并且⾃动维护对象的依赖关系。

## bean的装配方式

通过xml或Java代码的方式进行装配，官方推荐用`@Confinguration`加方法`@Bean`的方式进行装配

### 1、xml装配

比较传统的装配

1. xml编写

- 关于bean的name和id，id是唯一标识，name是别名，一个bean可用有多个name

- 配置文件中不允许出现两个id相同的，否则在初始化时即会报错

- 但配置文件中允许出现两个name相同的，在用getBean()返回实例时，后面一个Bean被返回，应该是前面那个被后面同名的 覆盖了为了避免不经意的同名覆盖的现象，尽量用id属性而不要用name属性。如果id和name都没有指定，则用类全名作为name，如，则你可以通过getBean("com.stamen.BeanLifeCycleImpl")返回该实例

2. 通过上下文获取

```java
ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
```

记得使用ClassPathXmlApplicationContext类创建上下文

### 2、@Configuration + @Bean

用@Configuration注释类，表明这个类是一个Bean的定义源
@Configuration允许调用同类中的其它@Bean方法来定义bean之间的关系，如下面的address中，调用了@Bean注释的user方法

一个被 @Configuration 标注的类，相当于一个 applicationContext.xml 的配置文件，这句话的意思就是，xml能做到的事情，可以用@Configuration 标注一个类来做到，除了@Bean外，还有很多的注解来对应xml中的标签

```java
@Configuration
public class AppConfig { 

    @Bean
    public User user() { 
        return new User(); 
    }

    @Bean
	public Address address() {
		return new Address(user());
	}
}
```

### 3、@ComponentScan + @Component

比较常用的方式，记得ComponentScan配置正确的包的扫描范围，否则报错找不到`BeanDefinition`

配置包扫描后，类被 @Component 注解标识后，类就被注入

包扫描配置示例: `@ComponentScan(basePackages="com.learn.spring.bean")`

spring boot的包扫描

```java
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
```

细心的人会发现，在我们单独使用这个注解的时候，需要配置扫描路径，而spring boot没有配置，其实注解只是标识，spring boot在自动装配中去做这个事情，它要取主启动类所在包及子包下的组件。就呼应了文档注释中的描述，也解释了为什么 SpringBoot 的启动器一定要在所有类的最外层

特别注意：

- 针对2和3的情况中，也就是用Java代码的方式装配，关于Configuration的作用
- 在2中，可以不用Configuration，也可以在3中，ComponentScan注解的类上加入Configuration，那么这个Configuration有什么作用？
- 在Configuration注解后，获取的bean都是同一个，也就是从缓存获取的

用@Configuration注解标注的类表明其主要目的是作为bean定义的源，@Configuration类允许通过调用同一类中的其他@Bean方法来定义bean之间的依赖关系

{% blockquote %}
补充 @Indexed 注解

https://www.cnblogs.com/aflyun/p/11992101.html 发现有这么一个东西，@Indexed注解，默认@Component就被@Indexed注解的，说是用了@Indexed，打包项目后读取META-INT/spring.components，不做包扫描，提高性能，不过要配置开启才行（我测试了确实如此）这个东西作用大吗？难道它这个转换比IOC反射效率高吗？还是强在省去了扫描步骤，该反射还是逃不过的
{% endblockquote %}

## BeanPostProcessor接口

bean的后置处理器，类可以实现该接口，可以让bean在创建的生命周期中的特定时间点，执行代码

```java
public class CatBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof Cat) {
            System.out.println("Cat postProcessBeforeInitialization run...");
        }
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof Cat) {
            System.out.println("Cat postProcessAfterInitialization run...");
        }
        return bean;
    }
    
}
```

## InitializingBean接口

当BeanFactory将bean创建成功，并设置完成所有它们的属性后，我们想在这个时候去做出自定义的反应，比如检查一些强制属性是否被设置成功，这个时候我们可以让我们的bean的class实现InitializingBean接口，以被触发
另一种替代实现InitializingBean的可选方案是在我们的bean的类内部定义一个init方法，然后在xml的bean定义中添加init属性即可触发调用

总结下来也就是下面的方式:

1. init-method(xml，指向一个方法) 或 @PostConstruct(注解在类的一个方法上)
2. InitializingBean接口，包含afterPropertiesSet方法

InitializingBean接口可以让bean在创建的生命周期中的特定时间点，执行代码

## XXXAware接口

感知接口，实现该接口的bean能获取到spring容器中记录该bean的一些属性，或者说让spring容器感知bean的存在

## Bean执行的顺序

初始化执行顺序：

构造方法

@PostConstruct / init-method
InitializingBean 的 afterPropertiesSet 方法

BeanPostProcessor的执行时机

before：构造方法之后，@PostConstruct之前
after：afterPropertiesSet之后



