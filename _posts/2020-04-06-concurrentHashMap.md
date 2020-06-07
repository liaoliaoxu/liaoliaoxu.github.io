---
layout:     post
title:      ConcurrentHashMap如何解决并发写入
subtitle:   ConcurrentHashMap如何解决并发写入
date:       2020-04-06
author:     Link
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - Java
---

# concurrentHashMap put方法处理并发

## concurrentHashMap存储结构

1. 首先是一个数组，用于通过hash值得到index，定位到某个node
2. 数组每个node都存着一个链表，或是红黑树
3. 插入时首先计算index，然后插入table[index]链表末尾/或者插入红黑树
4. 并发时锁的粒度在Node<k,v> f = table[index]，即锁住整个链表/红黑树

![table](/img/post-concurrent-node.png)

## put方法如何处理并发问题？

采取的方法是锁住链表头节点/红黑树根节点

![flow](/img/post-concurrent-flow.png)

## 与service-cache的异同

1. 在锁的粒度上没有太大的区别
   1. 都是用key算出index，然后锁住index代表的节点/loadingHolder
   2. 锁住的节点或者loadingholder实际上锁住了一组key(即计算index得到同一个值的所有key)
2. service-cache load是只让一个线程去load，然后写到缓冲区，让其他线程直接读
3. concurrentHashMap则是控制并发写入，只是做了加锁，最终还是会让每个线程都进入进行操作