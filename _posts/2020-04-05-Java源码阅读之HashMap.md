---
layout:     post   				    
title:      Java源码阅读之HashMap		
subtitle:   Java源码阅读之HashMap
date:       2020/04/05				
author:     Honey 					
header-img: img/post-girl.jpg 	
catalog: true 						
tags:								
    - Java
    - HashMap
---
## 成员变量

哈希数组的最大容量

```java
static final int MAXIMUM_CAPACITY = 1 << 30;
```

哈希数组默认容量

```java
static final int DEFAULT_INITIAL_CAPACITY = 16;
```

默认负载因子

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

链表节点数量大于此值时，从链表转化为红黑树

```java
static final int TREEIFY_THRESHOLD = 8;
```

数组容量大于此值时才会进行红黑树转换，也就是说要满足数组容量>64&链表节点数>8时才会从链表转换为红黑树

```java
static final int MIN_TREEIFY_CAPACITY = 64;
```

红黑树节点数量小于此值时，从红黑树转化为链表

```java
static final int UNTREEIFY_THRESHOLD = 6;
```

哈希数组

```java
transient Node<K,V>[] table;
```

元素个数

```java
transient int size;
```

### 插入过程

1. 判断哈希数组是否初始化，或者容量为0，则进行初始化。
2. 已经初始化，则计算出索引，找到待插入的hash槽。
3. 判断Hash槽是否有数据，没有数据则初始化一个Node，放在这。
4. 如果有数据，判断key是否相等，相等则直接替代。
5. 不等则判断是否为树节点，如果是树节点则插入树节点。
6. 不是树节点则向下判断，如果key相等则替代，遍历到末尾没有相同则创建一个Node插入链表中。
7. 判断链表长度是否大于8，大于则将链表转化为红黑树。
8. 最后判断一下插入之后容量是否大于阈值，若大于则进行扩容操作。

### 初始化中将容量设置为2的整数次幂

1. `Integer.numberOfLeadingZeros`返回二进制位中开头连续的0的个数
2. -1的二进制表示为`三十二个1`
3. 无符号右移之后加1就变成了大于该数值的最小2的整数次幂

```java
static final int tableSizeFor(int cap) {
    int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
    return (n<0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

### hash

hash时将key的hashcode的上16位和下16位进行一个异或（扰动函数），减少哈希冲突

```java
static final int hash(Object key) {
    int h;
    return (key == null)
        ? 0
        : (h = key.hashCode()) ^ (h >>> 16);
}
```

### resize

1. 扩容之后将链表分为两拨（树节点同理）
2. 由于扩容大小是以前的两倍，只用分析hash值上一位的值是否为1
3. 为0则索引不变，为1则加上原大小即可

```java
do {
    next = e.next;

    if((e.hash & oldCap) == 0) {
        if(loTail == null) {
            loHead = e;
        } else {
            loTail.next = e;
        }
        loTail = e;
    } else {
        if(hiTail == null) {
            hiHead = e;
        } else {
            hiTail.next = e;
        }
        hiTail = e;
    }
} while((e = next) != null);
```
