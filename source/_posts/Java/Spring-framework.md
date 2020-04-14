---
title: Spring Framework 与 Spring Boot
date: 2018-11-01 00:00:00
author: vanliuzh
top: true
cover: false
toc: true
mathjax: true
categories: Java
tags: [Java, Note]
reprintPolicy: cc_by
---

Spring Framework

<!-- more -->

## Ioc 设计理念

IoC(Inversion of Control)控制反转；依赖注⼊(dependency injection, DI)。它是⼀个对象定义依赖关系的过程，也就是说，对象只通过构造函数参数、⼯⼚⽅法的参数或对象实例构造或从⼯⼚⽅法返回后在对象实例上设置的属性来定义它们所使⽤的其他对象。

然后容器在创建bean时注⼊这些依赖项。这个过程基本上是bean的逆过程，因此称为控制反转(IoC) 在Spring中，构成应⽤程序主⼲并由Spring IoC容器管理的对象称为bean。

bean是由Spring IoC容器实例化、组装和管理的对象。IoC容器设计理念: 通过容器统⼀对象的构建⽅式，并且⾃动维护对象的依赖关系。

IOC流程: `装配`-->`解析`-->`注册`

本次笔记会比较乱，主要围绕各个散的知识点补充，然后就是`装配`-->`解析`-->`注册`这个流程，这个流程完成后，才能拿到bean，我们通常都是通过ApplicationContext来操作，不直接通过BeanFactory，下面是AnnotationConfigApplicationContext的构造器，传递的是Class对象，之后register(componentClasses)，注册，refresh()，刷新，得到上下文，bean可以获取到

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
		this();
		register(componentClasses);
		refresh();
	}
```

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

## 关于生命周期

如果bean 实现了BeanNameAware  接口，Spring  传递bean  的ID  到 setBeanName 方法。
如果 Bean 实现了 BeanFactoryAware 接口， Spring 传递beanfactory 给 setBeanFactory 方 法 。
如果有任何与 bean 相关联的 BeanPostProcessors，Spring 会在postProcesserBeforeInitialization()方法内调用它们。
如果 bean 实现 IntializingBean 了，调用它的 afterPropertySet 方法， 如果 bean 声明了初始化方法，调用此初始化方法。
如果有 BeanPostProcessors 和 bean 关联，这些 bean 的postProcessAfterInitialization() 方法将被调用。
如果 bean 实现了 DisposableBean，它将调用 destroy()方法。

## ioc 流程

内置bean和我们自己的bean

beanFactory

DefaultListableBeanFactory 工厂实现类，bean就在里面

注册内置bean给工厂，也就是加入 beanDefinitionMap 中

//  ConfigurationClassPostProcessor
//  AutowiredAnnotationBeanPostProcessor
//  CommonAnnotationBeanPostProcessor
// 	EventListenerMethodProcessor
// 	DefaultEventListenerFactory

添加2个后置处理器给工厂，这里是把class对象赋值到 beanFactory 的 beanPostProcessors 属性中，这个是一个CopyOnWriteArrayList类型的属性

ApplicationContextAwareProcessor, 
ApplicationListenerDetector


## 使用@Autowired注解警告Field injection is not recommended

这个警告的意思是 `使用变量依赖注入的方式是不被推荐的`，属于不严谨的警告

idea推荐 Always use constructor based dependency injection in your beans. Always use assertions for mandatory dependencies

意思是 `总是使用构造器的方式强制注入`

依赖注入有三种方式：

变量（filed）注入
构造器注入
set方法注入
工厂注入(好像是xml才有的)

```java
private RestTemplate restTemplate;

// set方法注入，不会有
@Autowired
public void setRestTemplate(RestTemplate restTemplate) {
    this.restTemplate = restTemplate;
}

// 这么写就会有告警
@Autowired
private ConsistentHashing consistentHashing;

// 构造器注入就不写了，和set注入类似，构造器注入可以不用@Autowired注解
```

其实这是一个很值得深挖的问题，因为我发现很多人都这么写，就是用变量注入。原因如下：

1. 这整个问题只是一个规范性的问题，但是好的规范肯定有它存在的意义，后面会说
2. 以前的idea是不会有这个问题的，新的版本有，说明spring在新版本中加强了规范

要讨论这个问题前，先说2个概念

1. 强制依赖: 在类被加载运行的时候就必须给他的依赖就是强制依赖，构造方法型依赖注入就是强制性，你初始化这个类就要给我这个依赖 

2. 选择依赖: 就是它在初始化的时候IOC不会给你注入，直到你使用到这个依赖时它才会把这个依赖给这个类，比如当你调用这个依赖的方法时才会给你把这个依赖给你注入进去，变量注入和set方法注入都是选择依赖

优点: 变量方式注入非常简洁，没有任何多余代码，非常有效的提高了java的简洁性。即使再多几个依赖一样能解决掉这个问题。

缺点: 不能有效的指明依赖。相信很多人都遇见过一个bug，依赖注入的对象为null，在启动依赖容器时遇到这个问题都是配置的依赖注入少了一个注解什么的，然而这种方式就过于依赖注入容器了，当没有启动整个依赖容器时，这个类就不能运转，在反射时无法提供这个类需要的依赖。
在使用set方式时，这是一种选择注入，可有可无，即使没有注入这个依赖，那么也不会影响整个类的运行。
在使用构造器方式时已经显式注明必须强制注入。通过强制指明依赖注入来保证这个类的运行。

另一个方面:
依赖注入的核心思想之一就是被容器管理的类不应该依赖被容器管理的依赖，换成白话来说就是如果这个类使用了依赖注入的类，那么这个类摆脱了这几个依赖必须也能正常运行。然而使用变量注入的方式是不能保证这点的。
既然使用了依赖注入方式，那么就表明这个类不再对这些依赖负责，这些都由容器管理，那么如何清楚的知道这个类需要哪些依赖呢？它就要使用set方法方式注入或者构造器注入

总结下: 变量方式注入应该尽量避免，使用set方式注入或者构造器注入，这两种方式的选择就要看这个类是强制依赖的话就用构造器方式，选择依赖的话就用set方法注入

看了上面的一些设计思想，再结合业务聊聊3个的区别

`变量注入和set注入可以不强制依赖，看起来好像这两个是一样，但是set注入有方法啊，可以在方法里面做判断，比如依赖不存在就做其它事情(给一个默认的实例等w)，这样整个类下面的代码逻辑就不会因为选择依赖问题而报错null，没错，说这么多，这句话就是核心，这也是为什么idea不推荐用变量注入，如果注入失败，类后面依赖这个变量的代码都有问题`

`好像依赖注入很好啊，为什么还有构造器注入呢？因为选择依赖不能暴露问题，如果你业务需求就是这个依赖没程序无法执行，那么构造器依赖就可以在程序运行的时候暴露出这个问题了`

`最后一点，是构造器依赖才能做到的，那就是声名final类型，final类型变量要求实例化的时候要赋值，方法注入和set注入都不能实现`

```java
private RestTemplate restTemplate;

@Autowired
public RrdtoolController(ConsistentHashing consistentHashing) {
    this.consistentHashing = consistentHashing;
}

@Autowired
public void setRestTemplate(RestTemplate restTemplate) {
    this.restTemplate = restTemplate;
}

private final ConsistentHashing consistentHashing;
```

参考原文链接：
https://blog.csdn.net/zhangjingao/article/details/81094529

https://blog.csdn.net/w57685321/article/details/86244753?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task

## 关于属性的含义与idea

可以点击进去查看源码注释

/Users/liuzhi/.m2/repository/org/springframework/boot/spring-boot-actuator-autoconfigure/2.2.2.RELEASE/spring-boot-actuator-autoconfigure-2.2.2.RELEASE.jar!/META-INF/spring-configuration-metadata.json

在这个文件中，有相关的描述，idea就是读取了这个json文件做到属性预读取，和错误判断，比如你写的属性不支持是依据这个json文件来的

另外这个json文件只是一部分，但是整个idea能够判断你的属性是否正确并给出提示，就是依赖于json文件

可以自定义json，让不支持的属性也能被idea识别，该方法可以用来对大型项目做管理，对自定义的属性有严格的规范，这样就能通过idea一眼判断出是否有人写了不符合的属性

## java -jar 和 直接main方法运行的区别

把jar解压后，查看META-INF下的jar信息文件

```
Manifest-Version: 1.0
Implementation-Title: kafka-demo
Implementation-Version: 0.0.1-SNAPSHOT
Start-Class: com.liuzhidream.kafka.demo.KafkaDemoApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Build-Jdk-Spec: 1.8
Spring-Boot-Version: 2.1.11.RELEASE
Created-By: Maven Archiver 3.4.0
Main-Class: org.springframework.boot.loader.JarLauncher
```

有一个`Start-Class`。jar包的规范是由Main-Class为入口的，如果我们写一个类打成jar，那个带main方法的类就会在`Main-Class`标注，注意这是jar包的规范。这里Main-Class是`org.springframework.boot.loader.JarLauncher`，说明启动类是org.springframework.boot.loader.JarLauncher，这和直接main运行有区别了，spring boot 的jar包是通过JarLauncher去调用Start-Class标注的我们自己的启动类`KafkaDemoApplication`来运行项目的

这是一个扩展点，如果遇到一些问题，可以查看jar包信息是否正确

如果想看源码，要引入spring-boot-loader依赖，`/Users/liuzhi/.m2/repository/org/springframework/boot/spring-boot-loader/2.1.11.RELEASE/spring-boot-loader-2.1.11.RELEASE.jar!/org/springframework/boot/loader/JarLauncher.class` main方法在这，感兴趣可以深入

主要聊的是一些启动方式

mvnspring‐boot:run

boot早期版本可能会是直接启动JarLauncher javaorg.springframework.boot.loader.JarLauncher

## 关于日志的使用和配置

首先先回顾和总结一下各个日志模块，以及它们之间的关系

Java提供的日志类是 `java.util.logging`，创建实例，然后调用对应日志级别的方法就能输出日志了

`Commons Logging`是一个第三方日志库，特点是它可以挂接不同的日志系统，并通过配置文件指定挂接的日志系统
默认情况下，Commons Loggin自动搜索并使用Log4j（Log4j是另一个流行的日志系统），如果没有找到Log4j，再使用JDK Logging

`Log4j` 聊到了一个Log4j，Log4j是Commons Logging的实现，可以说Commons Logging是一个接口

`SLF4J 和 Logback`，SLF4J类似于Commons Logging，也是一个日志接口，而Logback类似于Log4j，是一个日志的实现

spring boot中主要使用SLF4J 和 Logback

依赖是这个`spring-boot-starter-logging`，已经在父依赖中默认依赖了，所以不需要显示的依赖。日志很少去定制什么，我们关注如何配置就行了

通过spring properties配置或者xml配置，都是为了读取文件给到logback工厂类创建日志，由于配置内容很多，一般都是用xml做，所以最好不要在properties中去设置了，都在xml中设置

当然我们可以去配置日志配置文件的路径

```yaml
logging:
  # 指明日志配置文件路径，默认是resource下。从父pom所在的目录加载，可以统一全部的日志配置
  config: logs/logback-spring.xml

logging:
  # 指明日志配置文件路径，默认是resource下。从当前项目的classpath下加载，就是resource下
  config: classpath:logs/logback-spring.xml
```

### if标签的运用

如果要在xml配置中使用if标签需要加入下面的依赖

```xml
<dependency>
    <groupId>org.codehaus.janino</groupId>
    <artifactId>janino</artifactId>
    <version>2.6.1</version>
</dependency>
```

使用举例

```xml
<if condition='property("ifOpenConsol").contains("true")'>
<then>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>${logback.charset}</charset>
        </encoder>
        <!--字符串 System.out 或者 System.err ，默认 System.out -->
        <target>System.out</target>
    </appender>
</then>
</if>
```

### 如何使用

官方推荐使用的xml名字的格式为：logback-spring.xml而不是logback.xml，至于为什么，因为带spring后缀的可以使用`<springProfile>`这个标签，这样我们就可以一份日志配置满足不同环境，下面是配置示例，包含了详细的注释了

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出 -->
<!-- scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true -->
<!-- scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟 -->
<!-- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false -->
<configuration scan="true" scanPeriod="10 seconds" debug="false">
    <!--<include resource="org/springframework/boot/logging/logback/base.xml" />-->

    <!-- 用来设置上下文名称，每个logger都关联到logger上下文，默认上下文名称为default -->
    <!-- 可以使用<contextName>设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改(这里指的是日志配置有些参数修改后生效，但是不能改这个)-->
    <contextName>logback</contextName>
    <!-- spring变量的值通过springProperty注入给logging的context-->
    <springProperty scop="context" name="spring.application.name" source="spring.application.name" defaultValue="kafka-service"/>
    <!-- name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量 -->
    <property name="log.path" value="logs/${spring.application.name}"/>

    <!-- 彩色日志 -->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
    <!-- 其它主题风格 -->
    <!-- <property name="CONSOLE_LOG_PATTERN" value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}) %boldYellow([%thread]) %highlight(%-5level) %boldGreen(%logger{50}) - %msg%n" />-->
    <!--文件-黑白-->
    <!-- <property name="FILE_LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n" /> -->

    <!-- <appender>：负责写日志的组件，它有两个必要属性name和class。name指定appender名称，class指定appender的全限定名 -->
    <!-- class为ch.qos.logback.core.ConsoleAppender 把日志输出到控制台 -->
    <!-- class为ch.qos.logback.core.FileAppender 把日志添加到文件 -->
    <!-- class为ch.qos.logback.core.rolling.RollingFileAppender 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->

    <!--输出到控制台-->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>info</level>
        </filter>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>
    </appender>


    <!--输出到文件-->

    <!-- 时间滚动输出 level为 DEBUG 日志，本配置注释比较详细，作为配置参考 -->
    <!-- 日志归档就是先把日志输出到<file>${log.path}/log_debug.log</file>记录的位置，满足条件的时候，截取该部分归档到 -->
    <!-- <fileNamePattern>${log.path}/debug/log-debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>中 -->
    <appender name="DEBUG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">¬
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${log.path}/log_debug.log</file>
        <!-- 如果 true，事件被追加到现存文件尾部。如果 false，清空现存文件.默认为 true -->
        <append>true</append>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录，不同的class实现不同的策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志归档 -->
            <fileNamePattern>${log.path}/debug/log-debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录debug级别的，通过配置onMatch和onMismatch实现，匹配的留下，不匹配的拒绝，所以只留下debug基本的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>debug</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 时间滚动输出 level为 INFO 日志 -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${log.path}/log_info.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天日志归档路径以及格式 -->
            <fileNamePattern>${log.path}/info/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录info级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 时间滚动输出 level为 WARN 日志 -->
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${log.path}/log_warn.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/warn/log-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录warn级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 时间滚动输出 level为 ERROR 日志 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${log.path}/log_error.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/error/log-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录ERROR级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>


    <!-- <logger>用来设置某一个包或者具体的某一个类的日志打印级别、以及指定<appender> -->
    <!-- <logger>仅有一个name属性，一个可选的level和一个可选的addtivity属性 -->
    <!-- name:      用来指定受此logger约束的某一个包或者具体的某一个类 -->
    <!-- level:     用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF -->
    <!--            还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别 -->
    <!--            如果未设置此属性，那么当前logger将会继承上级的级别 -->
    <!-- addtivity: 是否向上级logger传递打印信息。默认是true-->

    <!--<logger name="org.springframework.web" level="info"/>-->
    <!--<logger name="org.springframework.scheduling.annotation.ScheduledAnnotationBeanPostProcessor" level="INFO"/>-->

    <!--
        使用mybatis的时候，sql语句是debug下才会打印，而这里我们只配置了info，所以想要查看sql语句的话，有以下两种操作：
        第一种把<root level="info">改成<root level="DEBUG">这样就会打印sql，不过这样日志那边会出现很多其他消息
        第二种就是单独给dao下目录配置debug模式，这样配置sql语句会打印，其他还是正常info级别
     -->

    <!--
        root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF
        不能设置为INHERITED或者同义词NULL。默认是DEBUG
        可以包含零个或多个元素，标识这个appender将会添加到这个logger
        就是root节点是基本，通过配置appender-ref把它们整合到一个logger中，或者单独配置logger
        <root>它也是<logger>元素，但是它是根logger,是所有<logger>的上级。只有一个level属性，因为name已经被命名为"root"，且已经是最上级了
    -->

    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="DEBUG_FILE"/>
        <appender-ref ref="INFO_FILE"/>
        <appender-ref ref="WARN_FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>

    <!--开发环境:打印控制台-->
    <springProfile name="dev">
        <logger name="com.liuzhidream.kafka.demo" level="debug"/>
    </springProfile>

    <!--生产环境:输出到文件-->
    <!--生产环境下，将此级别配置为适合的级别，以免日志文件太多或影响程序性能 -->
    <springProfile name="pro">
        <root level="info">
            <appender-ref ref="CONSOLE"/>
            <appender-ref ref="DEBUG_FILE"/>
            <appender-ref ref="INFO_FILE"/>
            <appender-ref ref="ERROR_FILE"/>
            <appender-ref ref="WARN_FILE"/>
        </root>
    </springProfile>

</configuration>
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

## refresh 方法

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        // 获得刷新的beanFactory
        // 对于AnnotationConfigApplicationContext，作用：
        // 1.调用org.springframework.context.support.GenericApplicationContext.refreshBeanFactory，
        // 只是指定了SerializationId
        // 2.直接返回beanFactory(不用创建，容器中已存在)
        //  对于ClassPathXmlApplicationContext，作用：
        // 1.调用AbstractRefreshableApplicationContext.refreshBeanFactory
        // 2.如果存在beanFactory，先销毁单例bean，关闭beanFactory，再创建beanFactory
        // 3.注册传入的spring的xml配置文件中配置的bean，注册到beanFactory
        // 4.将beanFactory赋值给容器，返回beanFactory

        // 这里就体现了注解和xml的区别，注解是先创建工厂，再注册bean，而xml是先把bean加载了，放到工厂中，再把准备好的工厂赋值给容器
        // 注解: 在容器启动之前就创建beanFactory
        // xml: 即容器启动过程中创建beanFactory

        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        // 准备bean工厂： 指定beanFactory的类加载器， 添加后置处理器，注册缺省环境bean等
        // beanFactory添加了2个后置处理器 ApplicationContextAwareProcessor, ApplicationListenerDetector (new )
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            // 空方法
            // 允许在上下文的子类中对beanFactory进行后处理
            // 比如 AbstractRefreshableWebApplicationContext.postProcessBeanFactory
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            // 1.通过beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class)
            //   拿到ConfigurationClassPostProcessor
            // 2.通过ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry，注册所有注解配置的bean
            // 注册的顺序： @ComponentScan>实现ImportSelector>方法bean>@ImportResource("spring.xml")
            //  > 实现 ImportBeanDefinitionRegistrar  (相对的顺序，都在同一个配置类上配置)
            // 3. 调用ConfigurationClassPostProcessor#postProcessBeanFactory
            //  增强@Configuration修饰的配置类  AppConfig--->AppConfig$$EnhancerBySpringCGLIB
            // (可以处理内部方法bean之间的调用，防止多例)
            //  添加了后置处理器 ConfigurationClassPostProcessor.ImportAwareBeanPostProcessor (new)
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            // 注册拦截bean创建的后置处理器：
            // 1.添加Spring自身的：  BeanPostProcessorChecker （new）  以及注册了beanDefinition的两个
            //  CommonAnnotationBeanPostProcessor AutowiredAnnotationBeanPostProcessor
            //  重新添加ApplicationListenerDetector(new ) ，删除旧的，移到处理器链末尾
            // 2.用户自定义的后置处理器
            // 注册了beanDefinition的会通过 beanFactory.getBean(ppName, BeanPostProcessor.class) 获取后置处理器
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            // 初始化事件多播器
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            // 空方法
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            // 实例化所有剩余的(非懒加载)单例。
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                        "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

## AbstractBeanDefinition

是BeanDefinition的实现类，除了bean的id和name，bean的各种属性可以在这里面找到

`org.springframework.beans.factory.support.AbstractBeanDefinition`

## DefaultListableBeanFactory 类

容器底层用DefaultListableBeanFactory，即实现了BeanDefinitionRegistry，又实现了BeanFactory

它是在我们实例化 AnnotationConfigApplicationContext 的时候，由构造器调用父类构造器去实例化的，主要是GenericApplicationContext

`public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry`

在构造器中实例化，所以上下文已经对beanFactory成员变量完成初始化了

```java
public GenericApplicationContext() {
		this.beanFactory = new DefaultListableBeanFactory();
	}
```

全类名: `org.springframework.beans.factory.support.DefaultListableBeanFactory`

BeanFactory接口的实现，有几个重要的属性和方法是需要关注的

该成员变量存储bean名称到definition的映射

```java
/** Map of bean definition objects, keyed by bean name. */
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
```

registerBeanDefinition用来向beanDefinitionMap注册beanDefinition，方法的逻辑不复杂，去掉各种校验核心代码也就几行

```java
	//---------------------------------------------------------------------
	// Implementation of BeanDefinitionRegistry interface
	//---------------------------------------------------------------------

	@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "Bean name must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");

		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}

		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
		if (existingDefinition != null) {
			if (!isAllowBeanDefinitionOverriding()) {
				throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
			}
			else if (existingDefinition.getRole() < beanDefinition.getRole()) {
				// e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
				if (logger.isInfoEnabled()) {
					logger.info("Overriding user-defined bean definition for bean '" + beanName +
							"' with a framework-generated bean definition: replacing [" +
							existingDefinition + "] with [" + beanDefinition + "]");
				}
			}
			else if (!beanDefinition.equals(existingDefinition)) {
				if (logger.isDebugEnabled()) {
					logger.debug("Overriding bean definition for bean '" + beanName +
							"' with a different definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			else {
				if (logger.isTraceEnabled()) {
					logger.trace("Overriding bean definition for bean '" + beanName +
							"' with an equivalent definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
			if (hasBeanCreationStarted()) {
				// Cannot modify startup-time collection elements anymore (for stable iteration)
				synchronized (this.beanDefinitionMap) {
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
					updatedDefinitions.addAll(this.beanDefinitionNames);
					updatedDefinitions.add(beanName);
					this.beanDefinitionNames = updatedDefinitions;
					removeManualSingletonName(beanName);
				}
			}
			else {
				// Still in startup registration phase
				this.beanDefinitionMap.put(beanName, beanDefinition);
				this.beanDefinitionNames.add(beanName);
				removeManualSingletonName(beanName);
			}
			this.frozenBeanDefinitionNames = null;
		}

		if (existingDefinition != null || containsSingleton(beanName)) {
			resetBeanDefinition(beanName);
		}
	}
```



