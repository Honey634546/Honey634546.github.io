---
layout:     post   				    
title:      Java源码阅读之HashMap		
subtitle:   Java源码阅读之HashMap
date:       2020/04/04 				
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

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        Node<K, V>[] tab;//指向当前Hash数组
        Node<K, V> p;//p指向Hash数组插入位置
        int n, i;//n为数组长度，i为索引
        
        //如果没有初始化或者长度为0，则初始化
        if((tab = table) == null || (n = tab.length) == 0) {

            tab = resize();
            
            n = tab.length;
        }
        
        p = tab[i = (n - 1) & hash];
        
        // 如果哈希槽为空，则在该槽上放置首个元素（普通Node）
        if(p == null) {
            tab[i] = newNode(hash, key, value, null);
            
            // 如果哈希槽不为空，则需要在哈希槽后面链接更多的元素
        } else {
            Node<K, V> e;
            K k;
            
            if(p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) {
                e = p;
                
                // 如果该哈希槽上链接的是红黑树结点，则需要调用红黑树的插入方法
            } else if(p instanceof TreeNode) {
                e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
            } else {
                // 遍历哈希槽后面链接的其他元素（binCount统计的是插入新元素之前遍历过的元素数量）
                for(int binCount = 0; ; ++binCount) {
                    // 如果没有找到同位元素，则需要插入新元素
                    if((e = p.next) == null) {
                        // 插入一个普通结点
                        p.next = newNode(hash, key, value, null);
                        
                        // 哈希槽（链）上的元素数量增加到TREEIFY_THRESHOLD后，这些元素进入波动期，即将从链表转换为红黑树
                        if(binCount >= TREEIFY_THRESHOLD - 1) { // -1 for 1st
                            treeifyBin(tab, hash);
                        }
                        
                        break;
                    }
                    
                    /*
                     * 对哈希槽后面链接的其他元素进行判断
                     *
                     * 只有哈希值一致（还说明不了key是否一致），且key也相同（必要时需要用到equals()方法）时，
                     * 这里才认定是存在同位元素（在HashMap中占据相同位置的元素）
                     */
                    if(e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                        break;
                    }
                    
                    p = e;
                }
            }
            
            // 如果存在同位元素（在HashMap中占据相同位置的元素）
            if(e != null) { // existing mapping for key
                // 获取旧元素的值
                V oldValue = e.value;
                
                // 如果不需要维持原状（可以覆盖旧值），或者旧值为null
                if(!onlyIfAbsent || oldValue == null) {
                    // 更新旧值
                    e.value = value;
                }
                
                afterNodeAccess(e);
                
                // 返回覆盖前的旧值
                return oldValue;
            }
        }
        
        // HashMap的更改次数加一
        ++modCount;
        
        // 如果哈希数组的容量已超过阈值，则需要对哈希数组扩容
        if(++size>threshold) {
            // 初始化哈希数组，或者对哈希数组扩容，返回新的哈希数组
            resize();
        }
        
        afterNodeInsertion(evict);
        
        // 如果插入的是全新的元素，在这里返回null
        return null;
    }
```