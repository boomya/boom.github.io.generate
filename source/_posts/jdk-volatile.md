title: 什么是volatile
date: 2016-02-01 20:48:43
categories: java
tags: ['java', 'volatile']
description:
---
什么是volatile仅仅用来保证该变量对所有线程的可见性，但不保证原子性
<!--more-->
### volatile概念
如果一个变量加了volatile关键字，就会告诉编译器和JVM的内存模型：这个变量是对所有线程共享的、可见的，每次jvm都会读取最新写入的值并使其最新值在所有CPU可见。

volatile 关键字的典型使用场景是在多线程环境下，多个线程共享变量，由于这些变量会缓存在 CPU 的缓存中，为了避免出现内存一致性错误而采用 volatile 关键字。
### volatile简单用法
##### 考虑下面这个生产者/消费者的例子
```java
public class ProducerConsumer {
  private String value = "";
  private boolean hasValue = false;
  public void produce(String value) {
    while (hasValue) {
      try {
        Thread.sleep(500);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
    System.out.println("Producing " + value + " as the next consumable");
    this.value = value;
    hasValue = true;
  }
  public String consume() {
    while (!hasValue) {
      try {
        Thread.sleep(500);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
    String value = this.value;
    hasValue = false;
    System.out.println("Consumed " + value);
    return value;
  }
}
```
在上面的类中，produce 方法通过存储参数来生成一个新的值，然后将 hasValue 设置为 true。while 循环检测标识变量（hasValue）是否 true，true 表示一个新的值没有被消费，要求当前线程睡眠（sleep），该睡眠一直循环直到标识变量 hasValue 变为 false，只有在新的值被 consume 方法消费完成后才能变为 false。如果没有有效的新值，consume 方法要求当前睡眠，当一个 produce
方法生成一个新值时，睡眠循环终止，并改变标识变量的值。

现在想象有两个线程在使用这个类的对象，一个生成值（写线程），另个一个消费值（读线程）。通过下面的测试来解释这种方式：
```java
public class ProducerConsumerTest {
  @Test
  public void testProduceConsume() throws InterruptedException {
    ProducerConsumer producerConsumer = new ProducerConsumer();
    List<String> values = Arrays.asList('1', '2', '3', '4', '5', '6', '7', '8',
        '9', '10', '11', '12', '13');
    Thread writerThread = new Thread(() -> values.stream()
        .forEach(producerConsumer::produce));
    Thread readerThread = new Thread(() -> {
      for (int i = 0; i < values.size(); i++) {
        producerConsumer.consume();
      }
    });
    writerThread.start();
    readerThread.start();
    writerThread.join();
    readerThread.join();
  }
}
```
这个例子大部分时候都能输出期望的结果，但是也有很大概率会出现死锁！

我们都知道计算机是由内存单元和 CPU （还有许多其他部分）组成。主内存就是程序指令、变量、数据存储的地方。程序执行期间，为了获得更好的性能，CPU 可能会将变量拷贝到自己的内存中（即所谓的 CPU 缓存）。由于现代计算机有多个 CPU，同样也存在多个 CPU 缓存。

在多线程环境下，有可能多个线程同时执行，每个线程使用不同的 CPU（虽然这完全依赖于底层的操作系统），每个 CPU 都从主内存中拷贝变量到它自己的缓存中。当一个线程访问这些变量时，是直接访问缓存中的副本，而不是真正访问主内存中的变量。

现在，假设在我们的测试中有两个线程运行在不同的 CPU 上，并且其中的有一个缓存了标识变量（或者两个都缓存了）。现在考虑如下的执行顺序

1、写线程生成一个值，并将 hasValue 设置为 true。但是只更新缓存中的值，而不是主内存。  
2、读线程尝试消费一个值，但是它的缓存副本中 hasValue 被设置为 false，所以即使写线程生产了一个新的值，也不能被消费，因为读线程无法跳出睡眠循环（hasValue 的值为 false）。  
3、因为读线程不能消费新生成的值，所以写线程也不能继续，因为标识变量没有设置回 false，因此写线程阻塞在睡眠循环中。  
4、这样，就产生了死锁！

这种情况只有在 hasValue 同步到所有缓存才能改变，这完全依赖于底层的操作系统。

如果我们将 hasValue 标示为 volatile，我就能确定这种死锁就不会再发生。
```java
private volatile boolean hasValue = false;
```
volatile 变量强制线程每次读取的时候都直接从主内存中读取，同时，每次写 volatile 变量的时候也要立即刷新主内存中的值。如果线程决定缓存变量，就需要每次读写的时候都与主内存进行同步。

做这个改变之后，我们再来考虑前面导致死锁的执行步骤

1、写线程生成一个值，并将 hasValue 设置为 true，这次直接更新主内存中的值（即使这个变量被缓存了）。  
2、读线程尝试消费一个值，先检查 hasValue 的值，每次读取都强制直接从主内存中获取值，所以能获取到写线程改变后的值。  
3、读线程消费完生成的值后，重新设置标识变量的值，这个新的值也会同步到主内存（如果这个值被缓存了，缓存的副本也会更新）。  
4、写线程获每次都是从主内存中取这个改变了的值，这样就能继续生成新的值。  

##### happens-before
强制线程直接从内存中读写线程，这是 Volatile 所能做全部的事情吗？
实际上，它还有更多的功能。访问一个 volatile 变量会在语句间建立 happens-before 关系。

什么是 happens-before 关系？  
happens-before 关系是程序语句之间的排序保证，这能确保任何内存的写，对其他语句都是可见的。

这与 volatile 是怎么关联的？  
当写一个 volatile 变量时，随后对该变量读时会创建一个 happens-before 关系。所以，所有在 volatile 变量写操作之前完成的写操作，将会对随后该 volatile 变量读操作之后的所有语句可见。

考虑下面这个例子
```java
// Definition: Some variables
// 变量定义
private int first = 1;
private int second = 2;
private int third = 3;
private volatile boolean hasValue = false;
// First Snippet: A sequence of write operations being executed by Thread 1
//片段 1：线程 1 顺序的写操作
first = 5;
second = 6;
third = 7;
hasValue = true;
// Second Snippet: A sequence of read operations being executed by Thread 2
//片段 2：线程 2 顺序的读操作
System.out.println("Flag is set to : " + hasValue);
System.out.println("First: " + first);  // will print 5 打印 5
System.out.println("Second: " + second); // will print 6 打印 6
System.out.println("Third: " + third);  // will print 7 打印 7
```
我们假设上面的两个代码片段有由两个线程执行：线程 1 和线程 2。当第一个线程改变 hasValue 的值时，它不仅仅是刷新这个改变的值到主存，也会引起前面三个值的写（之前任何的写操作）刷新到主存。结果，当第二个线程访问这三个变量的时候，就可以访问到被线程 1 写入的值，即使这些变量之前被缓存（这些缓存的副本都会被更新）。

这就是为什么我们不需要像第一个示例一样将变量标示为 volatile 。因为我们的写操作在访问 hasValue 之前，读操作在 hasValue 的读之后，它会自动与主内存同步。

还有另一个有趣的结论。JVM 因它的程序优化机制而闻名。有时对程序语句的重排序可以大幅度提高性能，并且不会改变程序的输出结果。例如，它可能会修改如语句的顺序：

```java
first = 5;
second = 6;
third = 7;
```
为
```java
second = 6;
third = 7;
first = 5;
```
但是，当多条语句涉及到对 volatile 变量的访问时，它永远不会将 volatile 变量前的写语句放在 volatile 变量之后，意思就是，它永远不会转换下列顺序：
```java
first = 5;  // write before volatile write //volatile 写之前的写
second = 6;  // write before volatile write //volatile 写之前的写
third = 7;   // write before volatile write //volatile 写之前的写
hasValue = true;
```
为
```java
first = 5;
second = 6;
hasValue = true;
third = 7;  // Order changed to appear after volatile write! This will never happen!
third = 7;  // 顺序发生了改变，出现在了 volatile 写之后。这永远不会发生。
```
volatile 变量会对性能有一定的影响。因为 volatile 变量强制访问主存，而访问主存肯定被访问 CPU 缓存慢。同时，它还防止 JVM 对程序的优化，这也会降低性能。
### 是否可以用volatile 变量来维护多线程之间的数据一致性
非常不幸，这是不行的。当多个线程读写同一个变量时，仅仅靠 volatile 是不足以保证一致性的.

volatile似乎是有时候可以代替简单的锁，似乎加了volatile关键字就省掉了锁。但又说volatile不能保证原子性（java程序员很熟悉这句话：volatile仅仅用来保证该变量对所有线程的可见性，但不保证原子性）。这不是互相矛盾吗？
#### volatile和atomic的区别
不要将volatile用在getAndOperate场合（这种场合不原子，需要再加锁），仅仅set或者get的场景是适合volatile的。

例如你让一个volatile的integer自增（i++），其实要分成3步：1）读取volatile变量值到local； 2）增加变量的值；3）把local的值写回，让其它的线程可见。这3步的jvm指令为：
```
mov    0xc(%r10),%r8d ; Load
inc    %r8d           ; Increment
mov    %r8d,0xc(%r10) ; Store
lock addl $0x0,(%rsp) ; StoreLoad Barrier
```
注意最后一步是内存屏障。
##### 什么是内存屏障
内存屏障（memory barrier）是一个CPU指令。基本上，它是这样一条指令： a) 确保一些特定操作执行的顺序； b) 影响一些数据的可见性(可能是某些指令执行后的结果)。编译器和CPU可以在保证输出结果一样的情况下对指令重排序，使性能得到优化。插入一个内存屏障，相当于告诉CPU和编译器先于这个命令的必须先执行，后于这个命令的必须后执行。内存屏障另一个作用是强制更新一次不同CPU的缓存。例如，一个写屏障会把这个屏障前写入的数据刷新到缓存，这样任何试图读取该数据的线程将得到最新值，而不用考虑到底是被哪个cpu核心或者哪颗CPU执行的。  

内存屏障（memory barrier）和volatile什么关系？上面的虚拟机指令里面有提到，如果你的字段是volatile，Java内存模型将在写操作后插入一个写屏障指令，在读操作前插入一个读屏障指令。这意味着如果你对一个volatile字段进行写操作，你必须知道：1、一旦你完成写入，任何访问这个字段的线程将会得到最新的值。2、在你写入前，会保证所有之前发生的事已经发生，并且任何更新过的数据值也是可见的，因为内存屏障会把之前的写入值都刷新到缓存。

#### volatile为什么没有原子性?
回到前面的JVM指令：从Load到store到内存屏障，一共4步，其中最后一步jvm让这个最新的变量的值在所有线程可见，也就是最后一步让所有的CPU内核都获得了最新的值，但中间的几步（从Load到Store）是不安全的，中间如果其他的CPU修改了值将会丢失。下面的测试代码可以实际测试voaltile的自增没有原子性：
```java
 private static volatile long _longVal = 0;

    private static class LoopVolatile implements Runnable {
        public void run() {
            long val = 0;
            while (val < 10000000L) {
                _longVal++;
                val++;
            }
        }
    }

    private static class LoopVolatile2 implements Runnable {
        public void run() {
            long val = 0;
            while (val < 10000000L) {
                _longVal++;
                val++;
            }
        }
    }

    private  void testVolatile(){
        Thread t1 = new Thread(new LoopVolatile());
        t1.start();

        Thread t2 = new Thread(new LoopVolatile2());
        t2.start();

        while (t1.isAlive() || t2.isAlive()) {
        }

        System.out.println("final val is: " + _longVal);
    }

Output:-------------

final val is: 11223828
final val is: 17567127
final val is: 12912109
```

#### 为什么AtomicXXX具有原子性和可见性？
就拿AtomicLong来说，它既解决了上述的volatile的原子性没有保证的问题，又具有可见性。它是如何做到的？就是CAS（比较并交换）指令。 其实AtomicLong的源码里也用到了volatile，但只是用来读取或写入，见源码：
```java
public class AtomicLong extends Number implements java.io.Serializable {
    private volatile long value;

    /**
     * Creates a new AtomicLong with the given initial value.
     *
     * @param initialValue the initial value
     */
    public AtomicLong(long initialValue) {
        value = initialValue;
    }

    /**
     * Creates a new AtomicLong with initial value {@code 0}.
     */
    public AtomicLong() {
    }
```
因为CAS是基于乐观锁的，也就是说当写入的时候，如果寄存器旧值已经不等于现值，说明有其他CPU在修改，那就继续尝试。所以这就保证了操作的原子性。
