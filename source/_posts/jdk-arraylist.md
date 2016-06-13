title: JDK-ArrayList
date: 2014-08-09 20:59:33
categories: JDK源码
tags: ['JDK', 'ArrayList']
description: ArrayList 是最常用的容器类,内部实现是看名字就知道是用的数组,一直再用从来没有看过内部的实现细节,今天学习一下.
---
ArrayList 是最常用的容器类,内部实现是看名字就知道是用的数组,一直再用从来没有看过内部的实现细节,今天学习一下.
<!--more-->
*ArrayList* 是最常用的容器类,内部实现是看名字就知道是用的数组,一直再用从来没有看过内部的实现细节,今天学习一下.
打开 *ArrayList* 最开始的代码
```java
private transient Object[] elementData;
```
果然是通过数组来保存数据.    
**transient** 是指明这个变量不要被序列化.

###1.关于遍历  
常用的遍历的方法就是创建迭代器
```java
Iterator<String> it = list.iterator();
while(it.hasNext()){
    String s = it.next();
}
```  
还有一种迭代器 *ListIterator* ,他继承了 *Iterator* ,除了有常规迭代器的方法还有 *hasPrevious()* 和 *previous()* 方法,提供了向前遍历.

###2.ConcurrentModificationException
在操作 *ArrayList* 时经常会发生 *ConcurrentModificationException* ,比如
```java
Iterator<String> it = list.iterator();
while(it.hasNext()){
    String s = it.next();
    list.remove(s); //list.add(s);
}
```
在遍历 *ArrayList* 过程中执行移除或添加的操作就会抛出 *ConcurrentModificationException* .在 *ArrayList* 在父类 *AbstractList* 有一个变量 *modCount* ,这个值在 *ArrayList* 执行移除或者添加方法时会执行 *modCount++* 操作,在创建迭代器时 `Iterator<String> it = list.iterator();` 会把 *modCount* 传入到迭代器中并保存一份副本 `int expectedModCount = modCount;`,迭代器在遍历过程中调用 *next()* 方法时会执行
```java
final void checkForComodification() {
	if (modCount != expectedModCount)
	    throw new ConcurrentModificationException();
    }
}
```
因为在遍历过程中执行了移除或者添加方法,改变了 *modCount* 的值,所以 *modCount* 和 *expectedModCount* 不匹配,抛出 *ConcurrentModificationException* .  
解决方案:  
```java
Iterator<String> it = list.iterator();
while(it.hasNext()){
    String s = it.next();
    it.remove(); 
}
```
将移除方法改为迭代器的移除方法,就不会抛出异常,因为 *it.remove()* 是通过当前的索引移除,并且在移除后会用最新的 *modCount* 更新 *expectedModCount* ,所以不会抛出异常.
还有一种解决方案,是在遍历过程中将需要移除掉的元素保存到一个临时的ArrayList中,在遍历完成后,执行 *removeAll* 方法. 

###3.自动扩容  
ArrayList是没有长度限制的,但是他内部的存储结构是数组,数组是需要固定长度的.ArrayList在添加一个元素的时候会判断是否已经大于当前数组的长度,如果大于,会创建一个新的数组,长度是当前数组长度的1.5倍,再将当前数据复制到新数组中.  

###4.UnsupportedOperationException
UnsupportedOperationException这个异常也很常见,在 *ArrayList* 的父类 *AbstractList* 定义了很多方法,方法体就是抛出 *UnsupportedOperationException* ,如果子类没有重写这个方法,调用的时候就会抛出这个异常
```java
public E remove(int index) {
    throw new UnsupportedOperationException();
}
```
这更像是一种编程技巧,可能会有这样一种场景,多个子类继承同一个父类,而子类之间的行为有可能不是全部相同或者说全部支持,通过上面这种方式处理,约定好一种 *exception* ,处理起来更加的优雅,易理解.