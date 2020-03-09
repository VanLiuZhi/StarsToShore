---
title: Java review Spring
date: 2019-04-05 00:00:00
tags: [java, note]
categories: Java
---

review

<!-- more -->

## 一致性哈希

先了解一个概念，取模和取余，如果除数都是正整数，那么取余和取模都是一样的，求余数
当 x 和 y 的正负号一样的时候，两个函数结果是等同的；当 x 和 y 的符号不同时，rem函数结果的符号和 x 的一样，而 mod 和 y 一样

在分布式中，或者用分库分表来说明，user表存在在3台机器上，用`用户id的hashcode % 3`来决定该用户的数据存储在哪台机器上。当机器增加到4台的时候，此时的模3就不对了，造成数据混乱。而一致性哈希算法就是解决这个问题的，一致性哈希保证当分布式环境的节点增加的时候，原来请求分配到的节点还是原来的节点

```java
public static void main(String[] args) {
    // 有5个用户
    Integer a = 123456;
    Integer b = 123457;
    Integer c = 123458;
    Integer d = 123459;
    Integer e = 123460;

    // 平均分配到3台机器
    System.out.println(a.hashCode() % 3);
    System.out.println(b.hashCode() % 3);
    System.out.println(c.hashCode() % 3);
    System.out.println(d.hashCode() % 3);
    System.out.println(e.hashCode() % 3);
    System.out.println("--------------");
    // 平均分配到4台机器
    System.out.println(a.hashCode() % 4);
    System.out.println(b.hashCode() % 4);
    System.out.println(c.hashCode() % 4);
    System.out.println(d.hashCode() % 4);
    System.out.println(e.hashCode() % 4);

    //        结果
    //        0
    //        1
    //        2
    //        0
    //        1
    //                --------------
    //        0
    //        1
    //        2
    //        3
    //        0

    // 对于d,e来说，增加机器的时候就错乱了
}
```

## http无状态

这句话体现在每个请求都是独立的，第二次的请求和第一次不会有关联

## 对称加密和非对称加密

1、加密和解密过程不同

对称加密过程和解密过程使用的同一个密钥，加密过程相当于用原文+密钥可以传输出密文，同时解密过程用密文-密钥可以推导出原文。但非对称加密采用了两个密钥，一般使用公钥进行加密，使用私钥进行解密。

2、加密解密速度不同

对称加密解密的速度比较快，适合数据比较长时的使用。非对称加密和解密花费的时间长、速度相对较慢，只适合对少量数据的使用。

3、传输的安全性不同

对称加密的过程中无法确保密钥被安全传递，密文在传输过程中是可能被第三方截获的，如果密码本也被第三方截获，则传输的密码信息将被第三方破获，安全性相对较低。
非对称加密算法中私钥是基于不同的算法生成不同的随机数，私钥通过一定的加密算法推导出公钥，但私钥到公钥的推导过程是单向的，也就是说公钥无法反推导出私钥。所以安全性较高。

## Spring boot 知识点收集

1. 配置: server.context-path 就是在请求上加个默认路径

2. @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})

排除此类的autoconfig，比如我们引入了mybatis的依赖，按照Spring的规范，就要装配对应的bean，没有配置数据库会报错。这个时候就能用这个注解来取消某些类的自动装配

3. @EnableDiscoveryClient与@EnableEurekaClient

都是服务client用的，如果用eureka之外的，用EnableDiscoveryClient，主要是为了组件通用化，可以通过注解参数取消服务注册，新版本可以去掉注解，具体看使用哪个版本

4. spring-cloud-config-client 和 spring-cloud-starter-config 都是配置依赖，区别是一个是starter风格的

5. @Component(组件) @Service(业务层) @Controller(web控制层) @Repository(持久层，持久层数据一般提供CURD) 都是组件，等效的，不过语义不一样

@Controller，装载的bean名称默认是类名首字母小写的名称，可以指定名称

@Service 用在一个接口的实现类上面

```java
@Service("UserDetailsService")
public class UserDetailsServiceImpl implements UserDetailsService {
    ...
}
```

@Service("UserDetailsService")注解是告诉Spring，当Spring要创建UserDetailsServiceImpl的的实例时，bean的名字必须叫做"UserDetailsService"，这样当Action需要使用UserServiceImpl的的实例时,就可以由Spring创建好的"UserDetailsService"，然后注入给Action：在Action只需要声明一个名字叫“UserDetailsService”的变量来接收由Spring注入的"UserDetailsService"即可，具体代码如下：

```java
// 注入UserDetailsService
@Resource(name = "UserDetailsService")
private UserDetailsService userDetailsService;
```

**@Resource的作用相当于@Autowired，只不过@Autowired按byType自动注入，而@Resource默认按 byName自动注入罢了**

@Repository对应数据访问层Bean

```java
@Repository(value="userDao")
public class UserDaoImpl extends BaseDaoImpl<User> {
    ...
}
```

6. @ConfigurationProperties(prefix = "mysql")

使用这个注解，可以把配置的属性装配到对象上，mysql是配置文件中的属性前缀，下面把url的属性拿到，实例对象的时候就用配置的属性去实例化

```java
@Data
@Configuration
@ConfigurationProperties(prefix = "mysql")
public class MysqlData {
    private String url;
//    private Integer port;
}

@Autowired
private MysqlData mysqlData;
```

7. @Primary @Qualifier

这两个注解都是用来对于Autowrite装配的时候，如果有多个实现，可以使用这两个注解
Qualifier接收一个参数指明实现的名称，和Autowrite一起使用，Primary只能存在一个(比如有2个实现，只能用在其中一个实现类上)，被注解的实现类优先用于Autowrite装配

8. @EnableAutoConfiguration

对自动装配这个点再做一些补充

要知道Spring的思想就是约定大于配置，这个注解在@SpringBootApplication下已经包含了，当我们使用pom引入依赖的时候，这些依赖的bean就会被装配的，主要就是自动装配发挥了作用，而约定就是这些依赖要按照一定的规范去开发，大家都遵守这些约定，才能被Spring装配到

getCandidateConfigurations  读取spring-boot项目的classpath下META-INF/spring.factories的内容

9. 修改系统服务的默认地址

例如在监控中: 如果对http://localhost:8030/hystrix 地址中的hystrix 小尾巴不满意怎么办？还记得Spring MVC的服务器端跳转（forward）吗？只需添加类似如下的Controller，就可以使用http://localhost:8030/ 访问到Hystrix Dashboard首页了。

```java
@Controller
public class HystrixIndexController {
  @GetMapping("")
  public String index() {
    return "forward:/hystrix";
  }
}
```

10. @PathVariable("patientId") Long id

把路由中匹配的参数在方法中形参重定义，下面的例子中，注解不传递参数，默认就是把形参和url参数对应，可以传参数然后重新定义形参

```java
@GetMapping("/get/{id}/{ex}")
    public ResponseBean<String> getCacheValue(@PathVariable String id, @PathVariable(value = "ex") String abc){
        String data = metricCacheService.getData(id);
        return new ResponseBean<>(data);
    }
```

11. profiles

```yml
profiles: dev
```

```yml
profiles:
  active: dev
```

要记住上面两种写法是不一样的，第二种是”激活“的意思，就是激活哪个profiles文件，而第一个是准备多个profiles文件的时候用

## 分布式事务

tcc lcn mq atomik seata

## 分布式session

Spring-boot-statrte-data-redis

## 静态化

velocity freemarker 模板引擎

## @Component 需不需要 @Autowired

该问题还没解决，我自定义一个普通的类，用component注解
`private MyUser myUser;` 去使用报空指针错误，必须要加上Autowired注解

如果我在构造器中

```java
public MyUser(RedisTemplate<String, Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
```

RedisTemplate是框架带的，加了这个后，可以不用Autowired注解访问

这个问题值得探讨，以前全都用Autowired，后来我发现有些人的不用也行，但是不是全部这样的，猜测是有依赖可以

