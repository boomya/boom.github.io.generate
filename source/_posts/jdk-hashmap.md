title: JDK-HashMap
date: 2014-08-10 20:30:11
categories: JDK源码
tags: ['JDK', 'HashMap']
description:
---
HashMap 也是一个经常用到的集合类, 有几个很基础的特性, 比方说key和value支持NULL, 不是同步安全, 可以自动扩容等.
<!--more-->
*HashMap* 也是一个经常用到的集合类, 有几个很基础的特性, 比方说key和value支持NULL, 不是同步安全, 可以自动扩容等. 我们现在来看一下具体的代码.  
###1. 存储结构
```java
/**
* The table, resized as necessary. Length MUST Always be a power of two.
*/
transient Entry<K,V>[] table;
```
*HashMap* 在内部维护着 *Entry* 类型的数组, 所有需要保存的元素都是存储在这里的. *Entry* 是 *HashMap* 一个内部静态类, 他的结构很类似于链表, 这也决定了当 *HashMap* 去 *put* 一个元素, 发生碰撞时就是通过链表去解决的,后面会有详细的描述.  
*HashMap* 在初始化的时候会初始化几个值, 初始大小, 扩充因子, 以及阀值, 扩充因子决定了当 *HashMap* 阀值的大小, 当 *HashMap* 的容量超过阀值时会触发扩容的方法,固定扩容到当前大小的2倍. *HashMap* 是有最大容量限制的, 是2^32.

###2. PUT
```java
public V put(K key, V value) {
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```
*HashMap* 单独对 *NULL* 的情况做了处理, 所以可以支持 *key* 或者 *value* 为 *NULL*. ***int hash = hash(key); int i = indexFor(hash, table.length);*** 这里计算这个 *key* 要放的位置的索引, 先是计算 *Hash值* , 再用 *Hash值* 和当前的长度做与运算计算出正确的位置(以前一直以为是取模原来是做的与运算).
还需要注意一点
```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }
    createEntry(hash, key, value, bucketIndex);
}
```
在 *PUT* 元素到 *HashMap* 之前会先做一次容量的判断, 如果当前的容量超过阀值会触发扩容方法, 扩容方法会做2件事, 1是将当前 *Entry* 数组的容易扩大为2倍, 2将当前的数据迁移到新的 *Entry* 数组中. 然后会重新计算要 *PUT* 的位置索引值. 在 *PUT* 时, 如果当前位置已经有一个 *Entry* 了, 会把这个 *Entry* 取出来当做新 *Entry* 的下一个元素, 形成了一个链表. 整个的 *PUT* 方法是没有同步锁的, 而且计算位置与插入数据是分开的步骤去做的, 所以在并发的情况, 如果有个多线程同时在写就会发生计算的位置不准确的问题, 后面会实例代码.

###3. GET
*GET* 方法相对没有那么复杂, 也是会对 *NULL* 进行特殊的处理. 在计算出位置的索引后, 如果取出的是一个链表还需要判断 *key* 是否相同.

###4. 并发
```java
public class App {
    static HashMap<Integer, Integer> map = new HashMap<Integer, Integer>(10);
    final static int COUNT = 10;
    final static CountDownLatch START_SIGNAL = new CountDownLatch(1);
    final static CountDownLatch END_SIGNAL = new CountDownLatch(COUNT);
    final static AtomicInteger count = new AtomicInteger(0);
    final static Runnable task = new Runnable(){
        @Override
        public void run() {
            try {
                START_SIGNAL.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            for(int i=0; i<10; i++){
                map.put(i, i);
            }
            count.incrementAndGet();
            END_SIGNAL.countDown();
        }
    };
    public static void main(String[] args){
        ExecutorService services = Executors.newFixedThreadPool(COUNT);
        for(int i=0; i<COUNT; i++){
            services.execute(task);
        }
        START_SIGNAL.countDown();
        try {
            END_SIGNAL.await();
            System.out.println(count.get() + " " + map.size());

            Set<Integer> keySet = map.keySet();
            for(Integer key:keySet){
                Integer value = map.get(key);
                System.out.println(key + " " + value);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (Exception ex){
            ex.printStackTrace();
        }
        services.shutdown();
    }
}
```
```
Result:
10 13
0 0
0 0
1 1
2 2
3 3
3 3
4 4
5 5
6 6
7 7
8 8
9 9
```
看到中间会出现重复的key, 这就是出现问题了, 如果换成 *Hashtable* 就不会出现这个问题, 因为 *Hashtable* 的 *PUT* 方法添加了修饰符 *synchronized*.
