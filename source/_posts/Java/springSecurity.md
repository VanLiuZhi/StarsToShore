---
title: springSecurity安全框架
date: 2019-07-28 00:00:00
author: vanliuzh
top: true
cover: false
toc: true
mathjax: false
summary: springSecurity安全框架 Spring Security is a powerful and highly customizable authentication and access-control framework. It is the de-facto standard for securing Spring-based applications.
categories: Java
tags: [Java, Note]
reprintPolicy: cc_by
---

springSecurity安全框架学习笔记，使用配置总结

<!-- more -->

## Security

如果要对Web资源进行保护，最好的办法莫过于Filter，要想对方法调用进行保护，最好的办法莫过于AOP

Security主要思想就是通过filter来实现的，整个流程包含各种过滤器

1. 主要过滤器  

WebAsyncManagerIntegrationFilter 
SecurityContextPersistenceFilter 
HeaderWriterFilter 
CorsFilter 
LogoutFilter
RequestCacheAwareFilter
SecurityContextHolderAwareRequestFilter
AnonymousAuthenticationFilter
SessionManagementFilter
ExceptionTranslationFilter
FilterSecurityInterceptor
UsernamePasswordAuthenticationFilter
BasicAuthenticationFilter        

2. 框架的核心组件

SecurityContextHolder 提供对SecurityContext的访问
SecurityContext 持有Authentication对象和其他可能需要的信息
AuthenticationManager 认证管理器，其中可以包含多个AuthenticationProvider
ProviderManager 对象为AuthenticationManager接口的实现类
AuthenticationProvider 主要用来进行认证操作的类 调用其中的authenticate()方法去进行认证操作
Authentication Spring Security方式的认证主体
GrantedAuthority 对认证主题的应用层面的授权，含当前用户的权限信息，通常使用角色表示
UserDetails 构建Authentication对象必须的信息，可以自定义，可能需要访问DB得到
UserDetailsService 通过username构建UserDetails对象，通过loadUserByUsername根据userName获取UserDetail对象 （可以在这里基于自身业务进行自定义的实现  如通过数据库，xml,缓存获取等）


## OAuth 2.0

OAuth 的核心就是向第三方应用颁发令牌，OAuth 2.0 规定了四种获得令牌的流程，可以选择一种方式颁发令牌

授权码（authorization-code）  一般前后端分离用这种
隐藏式（implicit）
密码式（password）
客户端凭证（client credentials）

`在程序中，需要做相关的配置，使用上面的英文`

注意，不管哪一种授权方式，第三方应用申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码：客户端 ID（client ID）和客户端密钥（client secret）。这是为了防止令牌被滥用，没有备案过的第三方应用，是不会拿到令牌的

`在程序中也有这个配置，声名哪些服务能获取令牌`


## UserDetailsService

这是一个接口，实现后需要返回一个框架自身的User类型对象，User又是UserDetails的实现

主要实现loadUserByUsername方法，就是如何通过用户名获取用户对象，这个用户对象必须是框架提供的类型，并且包括了相关的权限信息

```java
/**
 * @Description
 * @Author VanLiuZhi
 * @Date 2020-02-18 15:01
 */
@Service
public class MyUserDetailsUserService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        return new User(
                "User",
                PasswordEncoderFactories.createDelegatingPasswordEncoder().encode("123456"),
                AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
    }
}
```

上面是一个模拟，通过用户名加载到都是一样的User对象，实际应该去数据库查询

**关于密码加密的形式，我找到两种说法，这里做个记录。在我的比较新的版本中，已经不会出现方式一的问题了**

不过我也测试出了很多有价值的东西，在配置文件中配置`passwordEncoder`，使用`PasswordEncoderFactories.createDelegatingPasswordEncoder()`，和`userDetailsService`实现类中，也使用同样的(看上图的代码)，那么就是正常的。猜测原来默认不配置应该是使用`PasswordEncoderFactories.createDelegatingPasswordEncoder()`进行加密，后来改了，新版本的默认加密编码就用`BCryptPasswordEncoder()`

总结就是要统一，测试版本5.1.7，推荐使用方式二

```java
// config文件中
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userDetailsService).
            passwordEncoder(passwordEncoder());
}

@Bean
public PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```

{% blockquote %}

方式一:

不能直接创建 `BCryptPasswordEncoder` 对象来加密， 这种加密方式 没有 `{bcrypt}` 前缀
会导致在 matches 时导致获取不到加密的算法出现 `java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"`  问题。
问题原因是 Spring Security5 使用 DelegatingPasswordEncoder(委托)  替代 NoOpPasswordEncoder，并且 默认使用  BCryptPasswordEncoder 加密（注意 DelegatingPasswordEncoder 委托加密方法 BCryptPasswordEncoder 加密前 添加了加密类型的前缀）
https://blog.csdn.net/alinyua/article/details/80219500

方式二: 

1. 在配置中声名一个bean

@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

2. UserDetailsService实现类注入

@Autowired
private PasswordEncoder passwordEncoder;

3. 加密

passwordEncoder.encode("123456")
{% endblockquote %}

两种方式加密的结果，确实是多了个前缀，具体可以看源码

$2a$10$oUpt1tFddYCQXy0V3EEt6uM1V2BzRnYrGbcBhpOUFq1egzS2ocYZO
`{bcrypt}`$2a$10$CZWyE3FxdYfvYRGdTBVYue2i5PISShMxtgSnVJ4zs0RPyb9QLkJbW


## Security Config

Security配置，是继承覆盖父类的形式，围绕着两个方面来

1. 安全策略配置自定义
2. 用户详情和密码加密方法自定义

刚才对UserDetailService的实现在这里就用到了

### 用到的注解

`@EnableWebSecurity`
启用WebSecurity，都要配置

`@EnableGlobalMethodSecurity(prePostEnabled=true)`
要开启Spring方法级安全，在添加了@Configuration注解的类上再添加@EnableGlobalMethodSecurity注解即可，对请求方法做权限控制，一般都要用

查看注解，发现它还提供了很多类型

`prePostEnabled`： 确定 前置注解[@PreAuthorize,@PostAuthorize,..] 是否启用
`securedEnabled`： 确定安全注解 [@Secured] 是否启用
`jsr250Enabled`： 确定 JSR-250注解 [@RolesAllowed..]是否启用

可以启用多种类型的注解，但是在方法中只能生效一种

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true))
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    ...
}
```

在方法中使用，多数是在controller层用

```java
public interface UserService {
  List<User> findAllUsers();

  @PreAuthorize("hasAnyRole('user')")
  void updateUser(User user);

    // 下面不能设置两个注解，如果设置两个，只有其中一个生效
    // @PreAuthorize("hasAnyRole('user')")
  @Secured({ "ROLE_user", "ROLE_admin" })
  void deleteUser();
}
```

总结就是启用方法权限控制，通过设置的权限和用户的权限表做匹配决定是否能调用方法，这里用户的权限表就是通过UserDetailService来获取的。有三种模式，PreAuthorize用的比较广，有专门的表达式来编写，比如`hasAnyRole('user')`就是用了表达式，这个不是乱写的

参考
https://docs.spring.io/spring-security/site/docs/4.0.1.RELEASE/reference/htmlsingle/#el-common-built-in

https://www.jianshu.com/p/77b4835b6e8e


### 配置类

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailsService userDetailsService;

    /**
     * 这是Security的安全认证策略,默认的是所有请求都可以在授权之后访问
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .formLogin()
                .and()
                .authorizeRequests()
                .anyRequest()
                .authenticated();
    }
```

其它例子

```java
// 不会拦截/user,因为只会匹配"/api/**"，可以写多个http，不要冲突

http.requestMatchers().antMatchers("/api/**")
        .and().authorizeRequests().antMatchers("/user","/api/user").authenticated()
        .anyRequest().authenticated()
        .and().formLogin().loginPage("/login");
        //需要人证
        // http.authorizeRequests().antMatchers("/user").hasRole("Admin");
        // .and().formLogin().loginPage("/login");
        // api请求都不需要权限认证
        // http.authorizeRequests().antMatchers("/api").permitAll();
        // data/** Get请求不需要权限人证
        // http.authorizeRequests().antMatchers(HttpMethod.GET,"/data/**").permitAll();
```

方法使用说明，大致就是各种路由设置，鉴权设置

authorizeRequests() 开始请求权限配置
antMatchers() 使用Ant风格的路径匹配
permitAll() 用户可任意访问
anyRequest() 匹配所有路径
authenticated() 用户登录后可访问

- URL匹配
requestMatchers() 配置一个request Mather数组，参数为RequestMatcher 对象，其match 规则自定义，需要的时候放在最前面，对需要匹配的的规则进行自定义与过滤
authorizeRequests() URL权限配置
antMatchers() 配置一个request Mather 的 string数组，参数为 ant 路径格式， 直接匹配url
anyRequest 匹配任意url，无参 ,最好放在最后面

- 保护URL
authenticated() 保护UrL，需要用户登录
permitAll() 指定URL无需保护，一般应用与静态资源文件
hasRole(String role) 限制单个角色访问，角色将被增加 “ROLE_” .所以”ADMIN” 将和 “ROLE_ADMIN”进行比较. 另一个方法是hasAuthority(String authority)
hasAnyRole(String… roles) 允许多个角色访问. 另一个方法是hasAnyAuthority(String… authorities)
access(String attribute) 该方法使用 SPEL, 所以可以创建复杂的限制 例如如access("permitAll"), access("hasRole('ADMIN') and hasIpAddress('123.123.123.123')")
hasIpAddress(String ipaddressExpression) 限制IP地址或子网

- 登录login
formLogin() 基于表单登录
loginPage() 登录页
defaultSuccessUrl 登录成功后的默认处理页
failuerHandler登录失败之后的处理器
successHandler登录成功之后的处理器
failuerUrl登录失败之后系统转向的url，默认是this.loginPage + "?error"

- 登出logout
logoutUrl 登出url ， 默认是/logout， 它可以是一个ant path url
logoutSuccessUrl 登出成功后跳转的 url 默认是"/login?logout"
logoutSuccessHandler 登出成功处理器，设置后会把logoutSuccessUrl 置为null


特别

```java
// HttpSecurity 下的
public HttpSecurity and() {
    return HttpSecurity.this;
}
```

```java
// SecurityConfigurerAdapter 下的

public B and() {
    return getBuilder();
}

protected final B getBuilder() {
    if (securityBuilder == null) {
        throw new IllegalStateException("securityBuilder cannot be null");
    }
    return securityBuilder;
}
```

有2个and方法，有些方法会返回getBuilder，也和and一样，最后指向securityBuilder





