---
title: 'java 偏向锁、轻量级锁及重量级锁synchronized原理 '
date: 2020-03-16 09:55:34
updated: 2020-03-16 09:55:34
tags:
- java thread synchronized
categories:
- java
- thread
---
### synchronized的实现原理与应用
在多线程并发编程中synchronized一直是元老级角色，很多人都会称呼它为重量级锁。
Java SE 1.6中为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级 锁，以及锁的存储结构和升级过程。
先来看下利用synchronized实现同步的基础:Java中的每一个对象都可以作为锁。具体表现 为以下3种形式。
* 对于普通同步方法，锁是当前实例对象。
* 对于静态同步方法，锁是当前类的Class对象。
* 对于同步方法块，锁是Synchonized括号里配置的对象。
从JVM规范中可以看到Synchonized在JVM里的实现原理，JVM基于进入和退出Monitor对 象来实现方法同步和代码块同步，但两者的实现细节不一样。代码块同步是使用monitorenter 和monitorexit指令实现的，而方法同步是使用另外一种方式实现的，细节在JVM规范里并没有 详细说明。但是，方法的同步同样可以使用这两个指令来实现。

### Java对象头
synchronized用的锁是存在Java对象头里的。如果对象是数组类型，则虚拟机用3个字宽 (Word)存储对象头，如果对象是非数组类型，则用2字宽存储对象头。在32位虚拟机中，1字宽 等于4字节，即32bit。
![https](java_synchronized/markword00.png)
![https](java_synchronized/markword01.png)
在运行期间，Mark Word里存储的数据会随着锁标志位的变化而变化。Mark Word可能变 化为存储以下4种数据
![https](java_synchronized/markword02.png)
在64位虚拟机下，Mark Word是64bit大小的，其存储结构
![https](java_synchronized/markword03.png)

### 锁的升级与对比
锁一共有4种状态，级别从低到高依次是:无锁状态、偏向锁状态、轻量级锁状 态和重量级锁状态，这几个状态会随着竞争情况逐渐升级。锁可以升级但不能降级。
#### 偏向锁
##### 获取
HotSpot[1]的作者经过研究发现，大多数情况下，锁不仅不存在多线程竞争，而且总是由同 一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。当一个线程访问同步块并 获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出 同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头的Mark Word里是否 存储着指向当前线程的偏向锁。如果测试成功，表示线程已经获得了锁。如果测试失败，则需 要再测试一下Mark Word中偏向锁的标识是否设置成1(表示当前是偏向锁):如果没有设置，则 使用CAS竞争锁;如果设置了，则尝试使用CAS将对象头的偏向锁指向当前线程。
##### 释放
偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程尝试竞争偏向锁时， 持有偏向锁的线程才会释放锁。偏向锁的撤销，需要等待全局安全点(在这个时间点上没有正 在执行的字节码)。它会首先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否活着， 如果线程不处于活动状态，则将对象头设置成无锁状态;如果线程仍然活着，拥有偏向锁的栈 会被执行，遍历偏向对象的锁记录，栈中的锁记录和对象头的Mark Word要么重新偏向于其他 线程，要么恢复到无锁或者标记对象不适合作为偏向锁，最后唤醒暂停的线程。
![https](java_synchronized/pxlock.png)

#### 关闭偏向锁
偏向锁在Java 6和Java 7里是默认启用的，但是它在应用程序启动几秒钟之后才激活，如 有必要可以使用JVM参数来关闭延迟:-XX:BiasedLockingStartupDelay=0。如果你确定应用程 序里所有的锁通常情况下处于竞争状态，可以通过JVM参数关闭偏向锁:-XX:- UseBiasedLocking=false，那么程序默认会进入轻量级锁状态。

#### 轻量级锁
轻量级锁提升程序同步性能的依据是：对于绝大部分的锁，在整个同步周期内都是不存在竞争的（区别于偏向锁）。这是一个经验数据。如果没有竞争，轻量级锁使用CAS操作避免了使用互斥量的开销，但如果存在锁竞争，除了互斥量的开销外，还额外发生了CAS操作，因此在有竞争的情况下，轻量级锁比传统的重量级锁更慢。
* 轻量级锁加锁
线程在执行同步块之前，JVM会先在当前线程的栈桢中创建用于存储锁记录的空间，并 将对象头中的Mark Word复制到锁记录中，官方称为Displaced Mark Word。然后线程尝试使用 CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失 败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。
* 轻量级锁解锁
轻量级解锁时，会使用原子的CAS操作将Displaced Mark Word替换回到对象头，如果成 功，则表示没有竞争发生。如果失败，表示当前锁存在竞争，锁就会膨胀成重量级锁。图2-2是 两个线程同时争夺锁，导致锁膨胀的流程图。
![https](java_synchronized/qllock02.png)
![https](java_synchronized/qllock03.png)
![https](java_synchronized/qljlock.png)

#### 自旋锁与自适应自旋
Java的线程是映射到操作系统的原生线程之上的，如果要阻塞或唤醒一个线程，都需要操作系统来帮忙完成，这就需要从用户态转换到核心态中，因此状态转换需要耗费很多的处理器时间，对于代码简单的同步块（如被synchronized修饰的getter()和setter()方法），状态转换消耗的时间有可能比用户代码执行的时间还要长。
虚拟机的开发团队注意到在许多应用上，共享数据的锁定状态只会持续很短的一段时间，为了这段时间去挂起和恢复线程并不值得。如果物理机器有一个以上的处理器，能让两个或以上的线程同时并行执行，我们就可以让后面请求锁的那个线程“稍等一下“，但不放弃处理器的执行时间，看看持有锁的线程是否很快就会释放锁。为了让线程等待，我们只需让线程执行一个忙循环（自旋），这项技术就是所谓的自旋锁。
自旋锁在JDK1.4.2中引入，使用-XX:+UseSpinning来开启。JDK1.6中已经变为默认开。自旋等待不能代替阻塞。自旋等待本身虽然避免了线程切换的开销，但它是要占用处理器时间的，因此，如果锁被占用的时间很短，自旋等待的效果就会非常好，反之，如果锁被占用的时间很长，那么自旋的线程只会浪费处理器资源。因此，自旋等待的时间必须要有一定的限度，如果自旋超过了限定次数（默认是10次，可以使用-XX:PreBlockSpin来更改）没有成功获得锁，就应当使用传统的方式去挂起线程了。
JDK1.6中引入自适应的自旋锁，自适应意味着自旋的时间不在固定。而是有虚拟机对程序锁的监控与预测来设置自旋的次数。

自旋是在轻量级锁中使用的
### 锁的优缺点比较
![https](java_synchronized/lock_compare.png)

### 疑问思考
* jvm线程切换需要从用户态转换到核心态中，上线文切换,现场保存（如各个寄存器值保存，cpu缓存操作等）因此状态转换需要耗费很多的处理器时间。
重量级锁涉及到系统级线程切换，然而程序运行中大部分场景线程竞争不会太严重，或者资源持有时间很短，竞争线程稍等一会就能获取到锁；所以重量级锁性能比较差这个好理解。
* 对于偏向锁和轻量级锁都不涉及系统线程切换,为什么说偏向锁更轻量，性能更高呢？
这部分并没有资料提及，只能理解同样都是jvm实现的锁代码(对于jvm的C代码本人没有能力阅读，只能进行猜测)；轻量级锁流程简单代码效率更高。
偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程尝试竞争偏向锁时， 持有偏向锁的线程才会释放锁;只有没有竞争时效率才会更高


