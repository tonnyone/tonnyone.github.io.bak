---
title: java 并发01-synchoronized与volatile
category: java并发 
tag: java 
date: 2018-05-10 16:50:10
---

## synchoronized(内置锁)

内置锁的含义为： 在代码进入同步块的时候获取对象的锁，退出代码块后释放锁.
如果线程A尝试获取线程B的持有的锁，线程A必须等待或者阻塞.只有线程B释放锁(离开代码块或者抛出异常),线程A才能获得锁，如果线程B不释放锁,线程A将永远等待下去.

## volatile

volatile一般用作标记一个java变量**保存在主存**中,而不是缓存.这就意味着每一个读取volatile变量的线程是从主存读取,

<!--more-->
## syncoronized 的用法

- synchoronized 方法

1. 如果synchonized加在方法上，则锁定的是当前的对象(不可加在构造方法上面)
1. 如果synchonized加在静态方法上，锁定的是Class对象

- synchoronized 声明

获得括号内的对象的锁

```java
public class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }
}
```

如果count是Synchronized的一个实例对象,synchonized 有两层含义:

1. 保证了只有一个线程可以获得count的锁,所有其他的线程执行synchonized的方法都会挂起，直到一个线程执行完毕或者抛出异常,保证了**原子性**
1. 保证了当同步方法执行完毕后,synchronized关键字对count对象建立一个**happen-before**的关系,确保内存**可见性**

```java
public class Father{
    public synchronized void doTest(){

    }
}

public class son extends Father{
    public synchronized void doTest(){
        System.out.println("son do sometion");
        super.doTest();
    }
}
```

示例中son的doTest方法调用了父类的doTest()方法,son线程试图获取一个已经由它自己持有的锁,那么这个请求一定会成功.这就是锁的**重入性**,如果内置锁不是可重入的,那么上面代码会发生死锁,因为son的持有线程等待一个释放不了的锁。

## volatile 的用法

volatile变量提供了一种弱一点的同步机制.一般用于检查某个状态标记以判断是否推出循环.示例中,asleep必须为volatile变量.否则当asleep被另一个线程修改时,执行判断的线程却发现不了.

```java
volatile boolean asleep;
while(!asleep)
  countSomeSheep();
```

### 内存可见性

```java
 public class Puzzle {

    static boolean answerReady = false;
    static int answer = 0;
    static Thread t1 = new Thread() {
        public void run() {
            answer = 42;
            answerReady = true;
        }
    };
    static Thread t2 = new Thread() {
        public void run() {
            if (answerReady)
                System.out.println("The meaning of life is: " + answer);
            else {
                System.out.println("I don't know the answer");
            }
        };
    }

    public static void main(String[] args) throws InterruptedException {
        t1.start();
        t2.start();
        t1.join();
        t2.join();
    }
}
```

上面的代码可能输令人匪夷所思的结果: The meaning of life is: 0
当answerReady 为true的时候,answer 可能还是 0; 仿佛第6,7行跌倒了执行次序. 为了提升运行效率,编译器内存在不改变单线程执行结果的前提下,重新安排执行顺序:源代码---->编译器优化重排序---->指令级并行重排序---->内存系统重排序---->最终执行的指令序列.

volatile 主要语意保证了内存可见性,防止指令重排序

当且仅当满足以下条件时,才应该使用volatile变量：
1. 对变量的写入操作不依赖变量的当前值,或者你能确保只有单个线程更新变量的值.
1. 该变量不会与其他状态变量一起共同参与不变约束.
1. 访问变量时不需要加锁.

**注意:** 加锁机制既能确保可见性有可以确保原子性,而volatile变量只能确保可见性

## java 内存模型 

### synchonized 的实现

1. synchonized关键字经过编译会在同步代码块的前后形成**monitorenter**和**monitorexit**两个字节码指令
2. 当执行monitorenter指令的时候,首先尝试获取对象的锁，如果对象没有被锁定,或者当前线程已经拥有了对象的锁,则把锁的计数器加1
3. 在执行monitorexit指令的时候，会把锁的计数器减1,当计数器为0的时候锁就被释放

### volatile 的实现

1. volatile定义的变量保证了变量对所有线程的可见性,可见性是指当一条线程修改这个变量值的话,新值对于其他线程来说是立即可以得到的,而普通变量需要修改后再向主内存回写以后，才对其他变量可见.
2. volatile 第二个语义是禁止指令的重排序

- 普通变量仅仅会保证方法执行过程中所有依赖赋值结果的地方能获取到正确的结果.但是不能保证变量赋值操作的顺序与程序代码中执行的结果一致.
- volatile修饰的变量会再赋值后执行一个(local addl $0x0, (%esp))这个操作相当于一个内存屏障,指令从排序无法越过内存屏障

### 参考文献
- [oracle官网文档](https://docs.oracle.com/javase/tutorial/essential/concurrency/syncmeth.html)
- [java 并发编程实战](https://book.douban.com/subject/10484692/)
- [java 深入理解java虚拟机](https://book.douban.com/subject/24722612/)
- [知乎关于双重锁检查的讨论](https://www.zhihu.com/question/35268028) 





