# JMM和底层实现原理

## 一、线程之间的通信和同步

- 通信

  线程的通信是指线程之间以何种机制来交换信息。共享内存和消息传递

  1. 共享内存：

     在共享内存并发模型里，线程之间共享程序的公众状态，线程之间通过写-读内存中的公共状态来隐式进行通信，典型的共享内存通信方式就是通过共享对象进行通信

  2. 消息传递：

     在消息传递的并发模型里，线程没有公共状态，线程之间必须通过明确的发送消息来显式进行通信，在java中典型的消息传递方式就是wait()和notify().

- 同步

  同步指程序用于控制不同线程之间发生相对顺序的机制

  在共享内存并发模型里，同步是显式进行的。程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行。

  在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的。

## 二、Java内存模型（JMM）

![Java内存模型（JMM）](https://ws3.sinaimg.cn/large/005BYqpgly1g2bh7rgy81j30g306z76j.jpg)

### 内存屏障：禁止重排序

| 屏障类型            | 指令示例                 |                             说明                             |
| ------------------- | ------------------------ | :----------------------------------------------------------: |
| LoadLoad Barriers   | Load1;LoadLoad;Load2     |   确保Load1数据的装载，之前于Load2及所有后续装载指令的装载   |
| StoreStore Barriers | Store1;StoreStore;Store2 | 确保Store1数据对其他处理器可见（刷新到内存），之前于Store2及所有后续存储指令的存储 |
| LoadStore Barriers  | Load1;LoadStore;Store2   |  确保Load1数据的装载，之前于Store2及所有后续存储指令的存储   |
| StoreLoad Barriers  | Store1;StoreLoad;Load2   | 确保Store1数据对其他处理器可见（刷新到内存），之前于Load2及所有后续装载指令的装载。StoreLoad Barriers会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令。 |

### Happens-Before

#### 定义：

​		用happens-before的概念来阐述操作之间的内存可见性。在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系 。

​		两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second） 。

#### 加深理解：

​	1）如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。(对程序员来说)
 	2）两个操作之间存在happens-before关系，**并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行**。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么这种重排序是允许的(对编译器和处理器 来说)

#### Happens-Before规则：

1. 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
2. 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
3. volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
4. 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
5. start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
6. join() 规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。
7. 线程中断规则：对线程interrupt方法的调用happens-before于被中断线程的代码检测到中断事件的发生。

### volatile的内存语义

可以把对volatile变量的单个读/写，看成是使用同一个锁对这些单个读/写操作做了同步

#### volatile变量特性：

* 可见性：对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量**最后**的写入。

- 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。

### volatile写的内存语义：

* 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。

* 当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。

#### volatile重排序规则表

| 第一个操作 | 第二个操作 | 第二个操作 | 第二个操作 |
| ---------- | ---------- | :--------: | ---------- |
|            | 普通读/写  | volatile读 | volatile写 |
| 普通读/写  |            |            | 不允许     |
| volatile读 | 不允许     |   不允许   | 不允许     |
| volatile写 |            |   不允许   | 不允许     |

####  JMM对volatile的内存屏障插入策略

* 在每个volatile写操作的**前面**插入一个StoreStore屏障。在每个volatile写操作的**后面**插入一个StoreLoad屏障。

* u在每个volatile读操作的**后面**插入一个LoadLoad屏障。在每个volatile读操作的**后面**插入一个LoadStore屏障。

![](https://ws3.sinaimg.cn/large/005BYqpgly1g2bhtfh8e8j30ec09sab7.jpg)

![](https://ws3.sinaimg.cn/large/005BYqpgly1g2bhw67s86j30i70butao.jpg)

### 锁的内存语义

* 当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中

* 当线程获取锁时，JMM会把该线程对应的本地内存置为无效。从而使得被监视器保护的临界区代码必须从主内存中读取共享变量。

![](https://ws3.sinaimg.cn/large/005BYqpgly1g2bi0usl3kj30ec0c03yy.jpg)

![](https://ws3.sinaimg.cn/large/005BYqpggy1g2bi1eek8lj30gq0drjsn.jpg)

### final的内存语义

#### 1. 编译器和处理器要遵守两个重排序规则:

* 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序

* 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。

* **final域为引用类型**增加了如下规则：在构造函数内对一个final引用的对象的成员域的写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。

2. #### final语义在处理器中的实现

* 会要求编译器在final域的写之后，构造函数return之前插入一个StoreStore障屏。

* 读final域的重排序规则要求编译器在读final域的操作前面插入一个LoadLoad屏障

### volatile的实现原理

​      有volatile变量修饰的共享变量进行写操作的时候会使用CPU提供的Lock前缀指令

* 将当前处理器缓存行的数据写回到系统内存

* 这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效。

### synchronized的实现原理

#### 使用monitorenter和monitorexit指令实现的

* monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处

* 每个monitorenter必须有对应的monitorexit与之配对 

* 任何对象都有一个monitor与之关联，当且一个monitor被持有后，它将处于锁定状态 

#### 锁的存放位置

![](https://ws3.sinaimg.cn/large/005BYqpggy1g2bia4cki9j311r07kacp.jpg)

## 了解各种锁

锁一共有4种状态，级别从低到高依次是：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态

1. 偏向锁：

   大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。无竞争时不需要进行CAS操作来加锁和解锁

2. 轻量级锁

   无竞争时通过CAS操作来加锁和解锁。

3. 重量级锁

![](https://ws3.sinaimg.cn/large/005BYqpgly1g2bicpyxj8j30th05u41y.jpg)

