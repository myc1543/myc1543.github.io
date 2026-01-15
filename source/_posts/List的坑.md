---
title: List的坑
typora-root-url: List的坑
date: 2026-01-15 21:02:15
tags: [Java]
categories: [踩坑]
description: 记录一下使用List踩的坑
---

## Arrays.asList返回的List不支持增删操作

```
List<Integer> list = Arrays.asList(1,2,3);
list.add(4);
```

通过Arrays.asList得到的list插入数据， 此时会发生错误

```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractList.add(AbstractList.java:148)
	at java.util.AbstractList.add(AbstractList.java:108)
```



## 原因

```
private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
```

ArrayList不是util包下的ArrayList，而是Arrays的内部类，继承了AbstractList的抽象类，但是没有实现add、remove方法，调用AbstractList的方法。而AbstractList的方法是抛出一个异常。。。

```
public void add(int index, E element) {
        throw new UnsupportedOperationException();
    }
    
public E remove(int index) {
        throw new UnsupportedOperationException();
    }
```

