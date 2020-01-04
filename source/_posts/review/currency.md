## 基础与名称解释

竞态条件: 多个线程竞争同一个资源，如果对资源的访问顺序敏感，就称存在竞态条件
临界区: 导致竞态条件发生的代码称为临界区

Java运行至少启动两个线程，一个是main线程，一个是垃圾回收线程;Java启动的每一个进程都有一个独立的JVM

JMM通过控制主内存与每个线程的本地内存之间的交互，来为java程序员提供内存可见性保证(也就是线程A要向线程B发送消息，线程A要把数据先写到主内存，由线程B去主内存读取数据)

Java内存模型没有具体讲述前面讨论的执行策略是由编译器，CPU，缓存控制器还是其它机制促成的。甚至没有用开发人员所熟悉的类，对象及方法来讨论。取而代之，Java内存模型中仅仅定义了线程和内存之间那种抽象的关系。众所周知，每个线程都拥有自己的工作存储单元（缓存和寄存器的抽象）来存储线程当前使用的变量的值。Java内存模型仅仅保证了代码指令与变量操作的有序性，大多数规则都只是指出什么时候变量值应该在内存和线程工作内存之间传输。这些规则主要是为了解决如下三个相互牵连的问题：

1. 原子性：哪些指令必须是不可分割的。在Java内存模型中，这些规则需声明仅适用于-—实例变量和静态变量，也包括数组元素，但不包括方法中的局部变量-—的内存单元的简单读写操作。
2. 可见性：在哪些情况下，一个线程执行的结果对另一个线程是可见的。这里需要关心的结果有，写入的字段以及读取这个字段所看到的值。
3. 有序性：在什么情况下，某个线程的操作结果对其它线程来看是无序的。最主要的乱序执行问题主要表现在读写操作和赋值语句的相互执行顺序上。

## 并发编程模型的分类

`在并发编程中，我们需要处理两个关键问题：`

线程之间如何通信及线程之间如何同步（这里的线程是指并发执行的活动实体），通信是指线程之间以何种机制来交换信息。

`在命令式编程中，线程之间的通信机制有两种：共享内存和消息传递`

在共享内存的并发模型里，线程之间共享程序的公共状态，线程之间通过写-读内存中的公共状态来隐式进行通信。
在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过明确的发送消息来显式进行通信。

`同步是指程序用于控制不同线程之间操作发生相对顺序的机制`

在共享内存并发模型里，同步是显式进行的。程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行。
在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的。

## JMM

Java线程之间的通信由Java内存模型（简称为JMM）控制，JMM决定一个线程对共享变量的写入何时对另一个线程可见。

从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。

## 创建线程

- 实现Runnable接口
- 继承Thread类

```java
package currency;

public class MyThread implements Runnable {
    @Override
    public void run() {
        System.out.println("run method");
    }

    public static void main(String[] args) {
        Thread thread = new Thread(new MyThread());
        thread.start();
        System.out.println("线程唯一标识符：" + thread.getId());
        System.out.println("线程名称：" + thread.getName());
        System.out.println("线程状态：" + thread.getState());
        System.out.println("线程优先级：" + thread.getPriority());
    }
}

class MyThread2 extends Thread {

    @Override
    public void run() {
        System.out.println("run myThread2");
    }

    public static void main(String[] args) {
        MyThread2 myThread2 = new MyThread2();
        myThread2.start();
    }
}
```

## 线程优先级

从1到10，1最低，10最高

## 线程状态

new 新建
runnnable 可运行
blocked 阻塞
waiting 等待
time waiting 定时等待
terminated 终止

状态转换流程

1. 线程创建，进入new状态
2. 调用 start 或者 run 方法，进入 runnable 状态
3. JVM 按照线程优先级及时间分片等执行 runnable 状态的线程。开始执行时，进入 running 状态
4. 如果线程执行 sleep、wait、join，或者进入 IO 阻塞等。进入 wait 或者 blocked 状态
5. 线程执行完毕后，线程被线程队列移除。最后为 terminated 状态