---
title: Spring IOC 源码分析
date: 2019-11-01 00:00:00
tags: [java, note]
categories: Java
---

Spring IOC 源码分析

<!-- more -->

## Ioc 设计理念

IoC也称为依赖注⼊(dependency injection, DI)。它是⼀个对象定义依赖关系的过程，也就是说，对象只通过构造函数参数、⼯⼚⽅法的参数或对象实例构造或从⼯⼚⽅法返回后在对象实例上设置的属性来定义它们所使⽤的其他对象。

然后容器在创建bean时注⼊这些依赖项。这个过程基本上是bean的逆过程，因此称为控制反转(IoC) 在Spring中，构成应⽤程序主⼲并由Spring IoC容器管理的对象称为bean。

bean是由Spring IoC容器实例化、组装和管理的对象。IoC容器设计理念: 通过容器统⼀对象的构建⽅式，并且⾃动维护对象的依赖关系。

## bean的装配方式

通过xml或Java代码的方式进行装配，官方推荐用`@Confinguration`加方法`@Bean`的方式进行装配

### 1、xml装配

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

特别注意：

- 针对2和3的情况中，也就是用Java代码的方式装配，关于Configuration的作用
- 在2中，可以不用Configuration，也可以在3中，ComponentScan注解的类上加入Configuration，那么这个Configuration有什么作用？
- 在Configuration注解后，获取的bean都是同一个，也就是从缓存获取的

用@Configuration注解标注的类表明其主要目的是作为bean定义的源，@Configuration类允许通过调用同一类中的其他@Bean方法来定义bean之间的依赖关系

