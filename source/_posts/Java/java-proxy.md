---
title: Java Proxy 与 AOP
date: 2018-04-05 00:00:00
author: vanliuzh
top: true
cover: false
toc: true
mathjax: false
categories: Java
tags: [Java, Note]
reprintPolicy: cc_by
---

## 静态代理和动态代理

静态代理就是直接编码的形式，比如有一个UserDao，提供查询和修改数据库功能，假如我要扩展它，让每个数据库操作都在事务中执行，势必要修改代码。 可以代理这个UserDao，由代理类来处理事务，查询和修改的调用还是在UserDao中

参考 https://blog.csdn.net/hon_3y/article/details/70655966

例子

```java
package com.liuzhidream.rrdtool.store.service.proxy;

/**
 * @Description
 * @Author VanLiuZhi
 * @Date 2020-03-11 14:54
 */

// 服务接口
interface IUserDao {
    public int query(Integer pk);

    public int delete(Integer pk);
}

// 服务实现

class UserDao implements IUserDao {

    @Override
    public int query(Integer pk) {
        System.out.println("查询的数据id是" + pk);
        return pk;
    }

    @Override
    public int delete(Integer pk) {
        System.out.println("删除的数据id是" + pk);
        return pk;
    }
}

// 如果要扩展事务，需要修改代码
//public int delete(Integer pk) {
//    System.out.println("transaction staring");
//    System.out.println("删除的数据id是" + pk);
//    System.out.println("transaction staring");
//    return pk;
//}

// 使用静态代理

public class StaticProxy implements IUserDao {

    private UserDao userDao;

    public StaticProxy(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public int query(Integer pk) {
        System.out.println("transaction staring");
        int res = userDao.query(pk);
        System.out.println("transaction end");
        return res;
    }

    @Override
    public int delete(Integer pk) {
        System.out.println("transaction staring");
        int res = userDao.delete(pk);
        System.out.println("transaction end");
        return res;
    }
}
```

缺点:

1. 代理类也是对接口的实现，如果接口改了，也要跟着改
2. 代理对象需要实现和目标对象一样的接口，会有很多代理类，类太多

动态代理: 可以使用动态代理，代理对象，不需要实现接口，就不会有太多的代理类。动态代理是在内存中创建对象的，利用JDKAPI

动态代理约束: **目标对象一定是要有接口的，没有接口就不能实现动态代理**

动态代理举例，使用Proxy.newProxyInstance创建目标对象

```java
package com.liuzhidream.rrdtool.store.service.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * @Description
 * @Author VanLiuZhi
 * @Date 2020-03-11 14:02
 */
public class ProxyFactory {
    XiaoMing xiaoMing = new XiaoMing();

    public Person getProxy() {
        /*
         * loader 生成代理对象使用哪个类装载器【一般我们使用的是代理类的装载器】
         * interfaces 目标对象的接口。生成哪个对象的代理对象，通过接口指定【指定要代理类的接口】
         * h 生成的代理对象的方法里干什么事【实现handler接口，我们想怎么实现就怎么实现】
         *
         * public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
         */
        return (Person) Proxy.newProxyInstance(ProxyFactory.class.getClassLoader(), xiaoMing.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if (method.getName().equals("sing")) {
                    System.out.println("1000");
                    method.invoke(xiaoMing, args);
                }
                return null;
            }
        });
    }

    public static void main(String[] args) {
        ProxyFactory proxyFactory = new ProxyFactory();
        Person proxy = proxyFactory.getProxy();
        proxy.sing("你好");
    }

}

interface Person {
    void sing(String name);

    void dance(String name);
}

class XiaoMing implements Person {

    @Override
    public void sing(String name) {

        System.out.println("小明唱" + name);
    }

    @Override
    public void dance(String name) {

        System.out.println("小明跳" + name);

    }
}
```

## cglib

引入cglib – jar文件，spring core包含了对应的代码，可以用spring的

cglib在内存中动态构建子类

代理的类不能为final，否则报错(在内存中构建子类来做扩展，当然不能为final，有final就不能继承了)

目标对象的方法如果为final/static, 那么就不会被拦截，即不会执行目标对象额外的业务方法

代码举例

```java
package com.liuzhidream.rrdtool.store.service.proxy;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * @Description
 * @Author VanLiuZhi
 * @Date 2020-03-11 15:16
 */
public class CGlibProxyFactory implements MethodInterceptor {

    // 维护目标对象
    private Object target;

    public CGlibProxyFactory(Object target) {
        this.target = target;
    }

    // 给目标对象创建代理对象
    public Object getProxyInstance() {
        //1. 工具类
        Enhancer en = new Enhancer();
        //2. 设置父类(目标对象)
        en.setSuperclass(target.getClass());
        //3. 设置回调函数
        en.setCallback(this);
        //4. 创建子类(代理对象)
        return en.create();
    }


    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("开始事务.....");

        // 执行目标对象的方法
        Object returnValue = method.invoke(target, args);

        System.out.println("提交事务.....");
        return returnValue;
    }
}

class User {

    public int query(Integer pk) {
        System.out.println("query pk is" + pk);
        return pk;
    }

    public final int delete(Integer pk) {
        System.out.println("delete pk is" + pk);
        return pk;
    }

    public static void main(String[] args) {
        User user = (User) new CGlibProxyFactory(new User()).getProxyInstance();
        user.query(21);

        // class com.liuzhidream.rrdtool.store.service.proxy.User$$EnhancerByCGLIB$$fa2d90ed
        // 对象已经不是原对象了
        System.out.println(user.getClass()); 
        
        System.out.println(user.delete(21)); // 不会被拦截
    }
}

```

## 动态代理和cglib

动态代理是jdk通过的，cglib是asm字节码操作框架实现的。都是动态代理

但是jdk动态代理，`目标对象要实现接口`，有一定的局限性。cglib `目标对象不用实现接口`

## AOP

关注点代码和核心代码: 

- 所谓关注点代码就是重复的代码，就行最开始举例中的事务相关的代码，它的特点就是重复出现

- 核心代码就是业务代码，比如查询用户

使用AOP的作用: `关注点代码和核心代码分离`

这样做的好处: 1. 关注点代码只用写一次  2. 开发者把工作放到核心代码上 3. 运行时期，执行核心业务代码时候动态植入关注点代码，也就是代理