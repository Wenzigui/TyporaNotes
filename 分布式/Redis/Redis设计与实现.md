[toc]



# 简单动态字符串


## SDS结构

![image.png](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/1578896789840-c4c01623-0f08-4358-bac6-228d0a2f7b96.png)

```c
struct sdshdr {
    // SDS所保存字符串的长度，也等于buf数组中已使用字节的数量
	int len;
    // buf数组中未使用字节的数量
    int free;
    // 字节数组
    char buf[];
}
```

SDS遵循C字符串以空字符串结尾的惯例，遵循空字符结尾惯例的好处是，SDS可以直接重用一部分C字符串函数库里面的函数。


## C字符串的问题


### 获取字符串长度的性能问题

由于C字符串本身并不记录字符串的长度，所以每次获取一个C字符串的长度，程序必须遍历整个字符串，其时间复杂度为O(N)



<img src="https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/1578899111152-e5e82dd7-ed9d-4d44-997b-c553fae23cd9.png" alt="image.png" style="zoom: 50%;" />



SDS的len属性记录了SDS本身的长度，其时间复杂度为O(1)

通过SDS，Redis将获取字符串长度的**时间复杂度从O(N)降低到O(1)**，确保了获取字符串长度的工作不会成为Redis的性能瓶颈。


### 缓冲区溢出问题

由于C字符串不记录自身的长度，那么在执行某些函数对字符串进行修改的时候，可能会由于某个字符串没有足够的空间，导致对这个字符串的修改溢出到另外一个字符串，使得另外一个字符串被意外修改。



![image-20200520144222152](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200520144222152.png)



执行strcat函数，将"Redis"修改为"Redis Cluster"，但此时S1字符串并没有分配足够的空间，导致修改会溢出，波及S2字符串

![image.png](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/1578899689138-8c371160-7978-451d-a211-9a1a8d525a4f.png)

在使用SDS API的时候，会自动对SDS的空间进行检查、扩容等一系列操作，杜绝了缓冲区溢出的问题

> C字符串之于SDS，就像数组之于ArrayList



### 修改字符串频繁进行内存重新分配

每次增长或者缩短一个C字符串的时候，总是需要进行一次内存重新分配的操作

**频繁地内存重新分配会影响Redis的性能**

SDS通过**空间预分配**和**惰性空间释放**，减少需要执行的内存重新分配操作

空间预分配策略：

- 对SDS修改之后，如果SDS的长度小于1MB，那么将空间扩容为2 * len
- 对SDS修改之后，如果SDS的长度大于等于1MB，那么将空间扩容为len + 1MB

惰性空间释放：

当需要缩短SDS字符串长度的时候，程序并不立即通过内存重新分配对字符串进行缩容操作，而是通过free属性将多余的空间记录下来，等待将来使用



### 二进制安全问题

C字符串只能用于保存字符，为了确保Redis可以适用各种不同的使用场景，SDS API会以二进制的形式处理数据，不会对数据做任何限制，过滤。


## 总结

![image.png](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/1578900771323-ca0d02c5-58a0-494c-8a0b-b747e1dc0d55.png)



# Hash表 & 渐进式rehash

> 字典在redis中对的应用相当广泛，比如redis中的数据库就是使用字典来作为底层实现
> 对数据库的增、删、改、查操作也是构建在对字典的操作之上



## 字典的实现

Redis的字典底层使用哈希表实现，一个哈希表里面可以有多个哈希表节点，每个哈希表节点代表字典中的一个键值对

### 哈希表的结构

```c
typedef struct dictht {
    // 哈希表数组
	disEntry **table;
    
    // 哈希表的大小
    unsigned long size;
    
    // 哈希表的掩码，总是等于size - 1
    unsigned long sizemask;
  	
    // 哈希表中已有哈希节点的数量
    unsigned long used;
}
```

- table数组中每个元素都是一个指向dictEntry结构的指针
- 每个dictEntry结构都保存一个键值对
- size属性表示table数组的大小
- sizemask属性和哈希值一起决定一个键应该被放到table数组的那个index上
- used属性表示哈希表中已有的键值对数量

![image.png](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/1579007958165-ce155a08-6df5-48e5-b964-3b1c5065c174.png)
> 这是一个空的hash表

### 哈希节点的结构

```c
typedef struct dictEntry {
	// 键
    void *key;
    
    // 值
    union {
    	void *val;
        uint64_t u64;
        int64_t s64;
    }
    
    // 指向下一个哈希表节点的指针
    struct dictEntry *next;
}
```

- next属性是指向另一个哈希表节点的指针（通过链地址法解决哈希冲突）

![image.png](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/1579008251375-4be97a78-eeea-422e-bb6a-6d3bb6264394.png)

### 字典的结构

```c
typedef struct dict {
	// 类型特定函数
    dictType *type;
    
    // 私有数据
    void *privdata;
    
    // 哈希表
    dictht ht[2];
    
    // 用于rehash过程标识哈希表table需要进行rehash的index位置
    // 非rehash过程的时候，该属性的值为-1
    int rehashidx;
}
```

- type属性是一个指向dictType结构的指针，dictType结构保存了一簇用于操作特定类型键值对的函数
- privdata属性保存了需要传给那些类型特定的函数的可选参数
> 搞不懂上面这两个属性有啥用处

- ht是包含两个哈希表的数组
  - 字典日常操作只对ht[0]
  - ht[1]哈希表用于rehash操作

![image.png](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/1579008942797-0cb1e214-480d-4b44-bfa4-5b5fc4945105.png)



## rehash过程

> 为了让hash表的负载因子维持在一个合理的范围之内，当哈希表保存的键值对过多或者过少的时候，程序会对哈希表执行rehash操作

### 步骤

1. 为哈希表ht[1]分配合理的空间，扩容后哈希表的size必须是2的n次方
  1. 如果执行扩容操作，ht[1].size = ht[0].used * 2（ht[0].used = 15，那么ht[1].size = 32）
  1. 如果执行缩容操作，ht[1].size = ht[0].used（ht[0].used = 7，那么ht[1].size = 8）
2. 将ht[0]中所有的键值对rehash到ht[1]上
2. rehash结束后，ht[0] = ht[1]，为ht[1]新建一个空白的哈希表，为下一次rehash做准备
> rehash过程跟java的hashmap基本思路相同



### 扩容缩容的条件

扩容条件：

- 服务器目前没有执行gbsave命令或者bgrewriteaof命令，且哈希表的负载因子大于等于1
- 服务器目前正在执行gbsave命令或者bgrewriteaof命令，且哈希表的负载因子大于等于5

负载因子 = ht[0].used / ht[0].size

缩容条件：当哈希表的负载因子小于0.1的时候，程序自动开始对哈希表执行收缩操作



## 渐进式rehash

> redis对哈希表执行rehash操作并不是一次性的、集中式地完成；而是分多次，渐进式地完成
> 原因：一次性对一个庞大的哈希表（键值对以亿、万算）进行rehash会影响redis的性能

### 步骤

1. 为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两个哈希表
1. 字典中维持一个索引计数器rehashidx，将rehashidx设置为0，表示rehash工作正式开始
1. 在rehash期间，对字典的增删改查的操作的时候，会顺带将ht[0].table[rehashidx]位置上所有的键值对rehash到ht[1]，当rehash工作完成之后，程序将rehashidx属性+1
1. 随着字典的操作，最终在某个时间点上，ht[0]的所有键值对都会被rehash到ht[1]，此时程序将rehashidx设置为-1，整个字典的rehash操作完成



### 渐进式rehash的精髓

把整个字典的rehash工作均摊到对字典的增删改查操作上，从而避免集中式rehash操作带来的庞大计算量影响redis性能