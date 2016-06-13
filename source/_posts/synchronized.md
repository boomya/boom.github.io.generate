title: synchronized的简单用法
date: 2014-08-10 21:04:01
categories: java
tags: ['synchronized', 'java']
description:
---
简单说明下synchronized常用的场景以及用法
<!--more-->
synchronized可作用于instance变量、object reference（对象引用）、static函数和class literals(类名称字面常量).  
有几个需要注意的地方
- 无论synchronized关键字加在方法上还是对象上，他取得的锁都是对象，而不是把一段代码或函数当作锁,而且同步方法很可能还会被其他线程的对象访问。 
- 每个对象只有一个锁（lock）和之相关联。 
- 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。  

1. **把synchronized当作函数修饰符时**  
    ```java
    public synchronized void method(){   
        //......  
    } 
    ```
他锁定的是调用这个同步方法对象。也就是说，当一个对象P1在不同的线程中执行这个同步方法时，他们之间会形成互斥，达到同步的效果。但是这个对象所属的Class所产生的另一对象P2却能够任意调用这个被加了synchronized关键字的方法。  
上边的示例代码等同于如下代码：
```java
    public void method() {   
        synchronized (this){      //  (a)     
	        //......  
        }   
    }   
```
(a)处的this指的是什么呢？他指的就是调用这个方法的对象，如P1。可见同步方法实质是将synchronized作用于object reference。那个拿到了P1对象锁的线程，才能够调用P1的同步方法，而对P2而言，P1这个锁和他毫不相干，程式也可能在这种情形下摆脱同步机制的控制，造成数据混乱。  

2. **同步块**  
```java
    public void method(SomeObject so) {   
	    synchronized(so) {   
		    //......   
	    }   
    } 
```
这时，锁就是so这个对象，谁拿到这个锁谁就能够运行他所控制的那段代码。当有一个明确的对象作为锁时，就能够这样写程式，但当没有明确的对象作为锁，只是想让一段代码同步时，能够创建一个特别的instance变量（他得是个对象）来充当锁：
```java
    class Foo implements Runnable{   
        private byte[] lock = new byte[0];  // 特别的instance变量   
        public void method() {   
            synchronized(lock) { //...... }   
        }   
    	//......   
    }
```
**注：零长度的byte数组对象创建起来将比任何对象都经济――查看编译后的字节码：生成零长度的byte[]对象只需3条操作码，而Object lock = new Object()则需要7行操作码。**

3. **将synchronized作用于static函数**
```java
    class Foo {   
        public synchronized static void method1(){   // 同步的static 函数      
		    //......   
	    }   
	    public void method2(){   
            synchronized(Foo.class)   //  class literal(类名称字面常量)   
	    }   
    }   
```
&emsp;&emsp;代码中的method2()方法是把class literal作为锁的情况，他和同步的static函数产生的效果是相同的，取得的锁很特别，是当前调用这个方法的对象所属的类（Class，而不再是由这个Class产生的某个具体对象了）.  
&emsp;&emsp;记得在《Effective Java》一书中看到过将 Foo.class和 P1.getClass()用于作同步锁还不相同，不能用P1.getClass()来达到锁这个Class的目的。P1指的是由Foo类产生的对象。   
&emsp;&emsp;能够推断：假如一个类中定义了一个synchronized的static函数A，也定义了一个synchronized 的instance函数B，那么这个类的同一对象Obj在多线程中分别访问A和B两个方法时，不会构成同步，因为他们的锁都不相同。A方法的锁是Obj所属的那个Class，而B的锁是Obj所属的这个对象。
