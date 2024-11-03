---
title: Redis设计与实现
tags:
  - Redis
categories: [Database,Redis]
toc: true
date: 2022-03-03 12:02
updated: 2022-03-03 12:02
references:
  - title: 介绍 · Redis 设计与实现（第二版） · 看云
    url: https://www.kancloud.cn/kancloud/redisbook/63822
---

> 《Redis 设计与实现》一书全面而完整地讲解了 Redis 的内部运行机制， 对 Redis 的大多数单机功能以及所有多机功能的实现原理进行了介绍， 展示了这些功能的核心数据结构以及关键的算法思想。

<!-- more -->

## 底层数据结构

### 简单动态字符串

> - Redis 只会使用 C 字符串作为字面量， 在大多数情况下， Redis 使用 SDS （Simple Dynamic String，简单动态字符串）作为字符串表示。
> - 比起 C 字符串， SDS 具有以下优点：
> 	1. 常数复杂度获取字符串长度。
> 	2. 杜绝缓冲区溢出。
> 	3. 减少修改字符串长度时所需的内存重分配次数。
> 	4. 二进制安全。
> 	5. 兼容部分 C 字符串函数。

Redis 没有直接使用 C 语言传统的字符串表示（以空字符结尾的字符数组）， 而是自己构建了一种名为简单动态字符串（simple dynamic string，SDS）的抽象类型， 并将 SDS 用作 Redis 的默认字符串表示，是一个可以被修改的字符串值。

> 在 Redis 里面， C 字符串只会作为字符串字面量（string literal）， 用在一些无须对字符串值进行修改的地方， 比如打印日志

```shell
redis> SET msg "hello world"
OK
```

其中：

- 键值对的键是一个字符串对象， 对象的底层实现是一个保存着字符串 `"msg"` 的 SDS 。
- 键值对的值也是一个字符串对象， 对象的底层实现是一个保存着字符串 `"hello world"` 的 SDS 。

#### 底层实现

每个 `sds.h/sdshdr` 结构表示一个 SDS 值：

```c++
struct sdshdr {
    // 记录 buf 数组中已使用字节的数量
    // 等于 SDS 所保存字符串的长度
    int len;
    // 记录 buf 数组中未使用字节的数量
    int free;
    // 字节数组，用于保存字符串
    char buf[];
};
```

一个 SDS 示例：

- `free` 属性的值为 `0` ， 表示这个 SDS 没有分配任何未使用空间。
- `len` 属性的值为 `5` ， 表示这个 SDS 保存了一个五字节长的字符串。
- `buf` 属性是一个 `char` 类型的数组， 数组的前五个字节分别保存了 `'R'` 、 `'e'` 、 `'d'` 、 `'i'` 、 `'s'` 五个字符， 而最后一个字节则保存了空字符 `'\0'`（有需要时重用 `<string.h>` 函数库） 。

<img src="https://box.kancloud.cn/2015-09-13_55f50d7faffa3.png" alt="SDS"  />

> SDS 遵循 C 字符串以空字符结尾的惯例， 保存空字符的 `1` 字节空间不计算在 SDS 的 `len` 属性里面， 并且为空字符分配额外的 `1` 字节空间， 以及添加空字符到字符串末尾等操作都是由 SDS 函数自动完成的， 所以这个空字符对于 SDS 的使用者来说是完全透明的。
>
> 遵循空字符结尾这一惯例的好处是， SDS 可以直接重用一部分 C 字符串函数库里面的函数。
>
> 如果有个指针 s 指向 SDS，那么可以通过 `printf("%s", s->buf);` 打印

下面这个 SDS 为 `buf` 数组分配了五字节未使用空间， 所以它的 `free` 属性的值为 `5`

![未使用空间SDS](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203031221680.png)

#### SDS 与 C 字符串的区别

C 语言使用长度为 `N+1` 的字符数组来表示长度为 `N` 的字符串， 并且字符数组的最后一个元素总是空字符 `'\0'`

![C字符串](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203031223573.png)

以上并不能满足 Redis 对字符串在安全性、效率、以及功能方面的要求：

SDS 通过未使用空间解除了字符串长度和底层数组长度之间的关联：

在 SDS 中， `buf` 数组的长度不一定就是字符数量加一， 数组里面可以包含未使用的字节， 而这些字节的数量就由 SDS 的 `free` 属性记录。

> 对于 C 字符串：
>
> - 如果程序执行的是增长字符串的操作， 比如拼接操作（append）， 那么在执行这个操作之前， 程序需要先通过内存重分配来扩展底层数组的空间大小 —— 如果忘了这一步就会产生**缓冲区溢出**。
>
> - 如果程序执行的是缩短字符串的操作， 比如截断操作（trim）， 那么在执行这个操作之后， 程序需要通过内存重分配来释放字符串不再使用的那部分空间 —— 如果忘了这一步就会产生**内存泄漏**。
>
> Redis 作为数据库， 经常被用于速度要求严苛、数据被频繁修改的场合， 如果每次修改字符串的长度都需要执行一次内存重分配的话， 那么光是执行内存重分配的时间就会占去修改字符串所用时间的一大部分， 如果这种修改频繁地发生的话， 可能还会对性能造成影响。
>
> - **空间预分配**：优化 SDS 的字符串增长操作，程序不仅会为 SDS 分配修改所必须要的空间， 还会为 SDS 分配额外的未使用空间。
> - **惰性空间释放**：优化 SDS 的字符串缩短操作，程序并不立即使用内存重分配来回收缩短后多出来的字节， 而是使用 `free` 属性将这些字节的数量记录起来， 并等待将来使用。

<table><thead><tr><th>C 字符串</th><th>SDS</th></tr></thead><tbody><tr><td>获取字符串长度的复杂度为&nbsp;O(N)&nbsp;</td><td>获取字符串长度的复杂度为&nbsp;O(1)&nbsp;</td></tr><tr><td>API 是不安全的，可能会造成缓冲区溢出</td><td>API 是安全的，不会造成缓冲区溢出，自动将 SDS 的空间扩展至执行修改所需的大小</td></tr><tr><td>修改字符串长度&nbsp;<code>N</code>&nbsp; 次<b>必然</b>需要执行&nbsp;<code>N</code>&nbsp; 次内存重分配</td><td>修改字符串长度&nbsp;<code>N</code>&nbsp; 次<b>最多</b>需要执行&nbsp;<code>N</code>&nbsp; 次内存重分配。</td></tr><tr><td>只能保存文本数据</td><td>可以保存文本或者二进制数据，以处理二进制的方式来处理 SDS 存放在 buf 数组里的数据，</td></tr><tr><td>可以使用所有&nbsp;<code>&lt;string.h&gt;</code>&nbsp; 库中的函数</td><td>可以使用一部分&nbsp;<code>&lt;string.h&gt;</code>&nbsp; 库中的函数</td></tr></tbody></table>

### 链表

> - 链表被广泛用于实现 Redis 的各种功能， 比如列表键， 发布与订阅， 慢查询， 监视器， 等等。
> - 每个链表节点由一个 `listNode` 结构来表示， 每个节点都有一个指向前置节点和后置节点的指针， 所以 Redis 的链表实现是双端链表。
> - 每个链表使用一个 `list` 结构来表示， 这个结构带有表头节点指针、表尾节点指针、以及链表长度等信息。
> - 因为链表表头节点的前置节点和表尾节点的后置节点都指向 `NULL` ， 所以 Redis 的链表实现是无环链表。
> - 通过为链表设置不同的类型特定函数， Redis 的链表可以用于保存各种不同类型的值。

#### 底层实现

每个链表节点使用一个 `adlist.h/listNode` 结构来表示，即一个双向链表：

```c++
typedef struct listNode {
    // 前置节点
    struct listNode *prev;
    // 后置节点
    struct listNode *next;
    // 节点的值
    void *value;
} listNode;
```

使用 `adlist.h/list` 来持有链表的话， 操作起来会更方便：

```c++
typedef struct list {
    // 表头节点
    listNode *head;
    // 表尾节点
    listNode *tail;
    // 链表所包含的节点数量
    unsigned long len;
    // 节点值复制函数
    void *(*dup)(void *ptr);
    // 节点值释放函数
    void (*free)(void *ptr);
    // 节点值对比函数
    int (*match)(void *ptr, void *key);
} list;
```

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203031244834.png" alt="list 结构 " style="zoom: 80%;" />

#### 链表特性

Redis 的链表实现的特性可以总结如下：

- 双端： 链表节点带有 `prev` 和 `next` 指针， 获取某个节点的前置节点和后置节点的复杂度都是 $O(1)$。
- 无环： 表头节点的 `prev` 指针和表尾节点的 `next` 指针都指向 `NULL` ， 对链表的访问以 `NULL` 为终点。
- 带表头指针和表尾指针： 通过 `list` 结构的 `head` 指针和 `tail` 指针， 程序获取链表的表头节点和表尾节点的复杂度为 $O(1)$。
- 带链表长度计数器： 程序使用 `list` 结构的 `len` 属性来对 `list` 持有的链表节点进行计数， 程序获取链表中节点数量的复杂度为 $O(1)$。
- 多态： 链表节点使用 `void*` 指针来保存节点值， 并且可以通过 `list` 结构的 `dup` 、 `free` 、 `match` 三个属性为节点值设置类型特定函数， 所以链表可以用于保存各种不同类型的值。

### 字典

> - 字典被广泛用于实现 Redis 的各种功能， 其中包括数据库和哈希键。
> - Redis 中的字典使用**哈希表**作为底层实现， 每个字典带有两个哈希表， 一个用于平时使用， 另一个仅在进行 rehash 时使用。
> - 当字典被用作数据库的底层实现， 或者哈希键的底层实现时， Redis 使用 MurmurHash2 算法来计算键的哈希值。
> - 哈希表使用**链地址法**来解决键冲突， 被分配到同一个索引上的多个键值对会连接成一个单向链表。
> - 在对哈希表进行扩展或者收缩操作时， 程序需要将现有哈希表包含的所有键值对 rehash 到新哈希表里面， 并且这个 rehash 过程并不是一次性地完成的， 而是渐进式地完成的。

#### 底层实现

Redis 字典所使用的哈希表由 `dict.h/dictht` 结构定义：

```c++
typedef struct dictht {
    // 哈希表数组
    // 数组中的每个元素都是一个指向 dict.h/dictEntry 结构的指针， 每个 dictEntry 结构保存着一个键值对。
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;
    // 该哈希表已有节点的数量
    unsigned long used;
} dictht;
```

哈希表节点使用 `dictEntry` 结构表示， 每个 `dictEntry` 结构都保存着一个键值对：

```c++
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```

Redis 中的字典由 `dict.h/dict` 结构表示：

```c++
typedef struct dict {
    // 类型特定函数
    dictType *type;
    // 私有数据
    void *privdata;
    // 哈希表
    // 一般只使用ht[0]；ht[1] 哈希表只会在对 ht[0] 哈希表进行 rehash 时使用。
    dictht ht[2];
    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */
} dict;
```

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203052058867.png" alt="img" style="zoom: 67%;" />

#### 哈希算法

Redis 计算哈希值和索引值的方法如下：

```shell
# 使用字典设置的哈希函数，计算键 key 的哈希值
hash = dict->type->hashFunction(key);

# 使用哈希表的 sizemask 属性和哈希值，计算出索引值
# 根据情况不同， ht[x] 可以是 ht[0] 或者 ht[1]
index = hash & dict->ht[x].sizemask;

# 假设计算得出的哈希值为 8 ， 那么程序会继续使用语句：
index = hash & dict->ht[0].sizemask = 8 & 3 = 0;
```

<table><tr>
	<td><img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203052105563.png" /></td>
	<td><img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203052104825.png" /></td>
</tr></table>

> 当字典被用作数据库的底层实现， 或者哈希键的底层实现时， Redis 使用 MurmurHash2 算法来计算键的哈希值。
>
> MurmurHash 算法最初由 Austin Appleby 于 2008 年发明， 这种算法的优点在于， 即使输入的键是有规律的， 算法仍能给出一个很好的随机分布性， 并且算法的计算速度也非常快。
>
> MurmurHash 算法目前的最新版本为 MurmurHash3 ， 而 Redis 使用的是 MurmurHash2 ， 关于 MurmurHash 算法的更多信息可以参考该算法的主页： <<http://code.google.com/p/smhasher/>> 。

#### 哈希冲突

Redis 的哈希表使用链地址法（separate chaining）（拉链法）来解决键冲突。

每个哈希表节点都有一个 `next` 指针， 多个哈希表节点可以用 `next` 指针构成一个单向链表， 被分配到同一个索引上的多个节点可以用这个单向链表连接起来， 这就解决了键冲突的问题。

因为 `dictEntry` 节点组成的链表没有指向链表表尾的指针， 所以为了速度考虑， 程序总是将新节点添加到链表的表头位置（复杂度为 $O(1)$)， 排在其他已有节点的前面。

<img src="https://box.kancloud.cn/2015-09-13_55f512c4d7f99.png" alt="img" style="zoom:67%;" />

#### Rehash

随着操作的不断执行， 哈希表保存的键值对会逐渐地增多或者减少， 为了让哈希表的负载因子（load factor）维持在一个合理的范围之内， 当哈希表保存的键值对数量太多或者太少时， 程序需要对哈希表的大小进行相应的扩展或者收缩。

扩展和收缩哈希表的工作可以通过执行 **rehash （重新散列）**操作来完成， Redis 对字典的哈希表执行 rehash 的步骤如下：

1. 为字典的 `ht[1]` 哈希表分配空间， 这个哈希表的空间大小取决于要执行的操作， 以及 `ht[0]` 当前包含的键值对数量 （也即是 `ht[0].used` 属性的值）：

		```
		 - 如果执行的是扩展操作， 那么 `ht[1]` 的大小为第一个大于等于 `ht[0].used * 2` 的 $2^n$（`2` 的 `n` 次方幂）；
		 - 如果执行的是收缩操作， 那么 `ht[1]` 的大小为第一个大于等于 `ht[0].used` 的 $2^n$。
		```

2. 将保存在 `ht[0]` 中的所有键值对 rehash 到 `ht[1]` 上面： rehash 指的是**重新计算**键的哈希值和索引值， 然后将键值对放置到 `ht[1]` 哈希表的指定位置上。
3. 当 `ht[0]` 包含的所有键值对都迁移到了 `ht[1]` 之后 （`ht[0]` 变为空表）， 释放 `ht[0]` ， 将 `ht[1]` 设置为 `ht[0]` ， 并在 `ht[1]` 新创建一个空白哈希表， 为下一次 rehash 做准备。

当以下条件中的任意一个被满足时， 程序会自动开始对哈希表执行扩展操作：

1. 服务器目前没有在执行 BGSAVE 命令或者 BGREWRITEAOF 命令， 并且哈希表的负载因子大于等于 `1`
2. 服务器目前正在执行 BGSAVE 命令或者 BGREWRITEAOF 命令， 并且哈希表的负载因子大于等于 `5`

其中哈希表的负载因子可以通过公式：**负载因子 = 哈希表已保存节点数量 / 哈希表大小**

$load\_factor = ht[0].used  /  ht[0].size$

> 在执行 BGSAVE 命令或 BGREWRITEAOF 命令的过程中， Redis 需要创建当前服务器进程的子进程， 而大多数操作系统都采用写时复制（[copy-on-write](http://en.wikipedia.org/wiki/Copy-on-write)）技术来优化子进程的使用效率。
>
> 所以在子进程存在期间， 服务器会提高执行扩展操作所需的负载因子， 从而尽可能地避免在子进程存在期间进行哈希表扩展操作， 这可以避免不必要的内存写入操作， 最大限度地节约内存。
>
> 另一方面， 当哈希表的负载因子小于 `0.1` 时， 程序自动开始对哈希表执行收缩操作。

#### 渐进式 Rehash

> 如果哈希表里保存的键值对数量是四百万、四千万甚至四亿个键值对， 那么要一次性将这些键值对全部 rehash 到 `ht[1]` 的话， 庞大的计算量可能会导致服务器在一段时间内停止服务。

为了避免 rehash 对服务器性能造成影响， 服务器不是一次性将 `ht[0]` 里面的所有键值对全部 rehash 到 `ht[1]` ， 而是分多次、渐进式地将 `ht[0]` 里面的键值对慢慢地 rehash 到 `ht[1]` 。

以下是哈希表渐进式 rehash 的详细步骤：

1. 为 `ht[1]` 分配空间， 让字典同时持有 `ht[0]` 和 `ht[1]` 两个哈希表。
2. 在字典中维持一个索引计数器变量 `rehashidx` ， 并将它的值设置为 `0` ， 表示 rehash 工作正式开始。
3. 在 rehash 进行期间， 每次对字典执行添加、删除、查找或者更新操作时， 程序除了执行指定的操作以外， 还会顺带将 `ht[0]` 哈希表在 `rehashidx` 索引上的所有键值对 rehash 到 `ht[1]` ， 当 rehash 工作完成之后， 程序将 `rehashidx` 属性的值增一。
4. 随着字典操作的不断执行， 最终在某个时间点上， `ht[0]` 的所有键值对都会被 rehash 至 `ht[1]` ， 这时程序将 `rehashidx` 属性的值设为 `-1` ， 表示 rehash 操作已完成。

> 在进行渐进式 rehash 的过程中， 字典会同时使用 `ht[0]` 和 `ht[1]` 两个哈希表， 所以在渐进式 rehash 进行期间， 字典的删除（delete）、查找（find）、更新（update）等操作会在两个哈希表上进行：
>
> - 要在字典里面查找一个键的话， 程序会先在 `ht[0]` 里面进行查找， 如果没找到的话， 就会继续到 `ht[1]` 里面进行查找， 诸如此类。
>
> - 在渐进式 rehash 执行期间， 新添加到字典的键值对一律会被保存到 `ht[1]` 里面， 而 `ht[0]` 则不再进行任何添加操作： 这一措施保证了 `ht[0]` 包含的键值对数量会只减不增， 并随着 rehash 操作的执行而最终变成空表。

### 跳表

> - 跳跃表是**有序集合**的底层实现之一， 除此之外它在 Redis 中没有其他应用。
> - Redis 的跳跃表实现由 `zskiplist` 和 `zskiplistNode` 两个结构组成， 其中 `zskiplist` 用于保存跳跃表信息（比如表头节点、表尾节点、长度）， 而 `zskiplistNode` 则用于表示跳跃表节点。
> - 每个跳跃表节点的层高都是 `1` 至 `32` 之间的随机数。
> - 在同一个跳跃表中， 多个节点可以包含相同的分值， 但每个节点的成员对象必须是唯一的。
> - 跳跃表中的节点按照分值大小进行排序， 当分值相同时， 节点按照成员对象的大小进行排序。

#### 底层实现

跳跃表节点的实现由 `redis.h/zskiplistNode` 结构定义：

```c++
typedef struct zskiplistNode {
    // 后退指针
    struct zskiplistNode *backward;
    // 分值，节点按各自所保存的分值从小到大排列
    double score;
    // 成员对象
    robj *obj;
    // 层
    struct zskiplistLevel {
        // 前进指针：访问位于表尾方向的其他节点
        struct zskiplistNode *forward;
        // 跨度：记录了前进指针所指向节点和当前节点的距离
        unsigned int span;
    } level[];
} zskiplistNode;
```

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203052123925.png" style="zoom: 67%;" />

`zskiplist` 结构的定义如下：

```c++
typedef struct zskiplist {
    // 表头节点和表尾节点
    struct zskiplistNode *header, *tail;
    // 表中节点的数量
    unsigned long length;
    // 表中层数最大的节点的层数
    int level;
} zskiplist;
```
