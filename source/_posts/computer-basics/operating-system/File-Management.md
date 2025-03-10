---
title: 文件管理
tags:
  - File
categories: [Computer Basics,Operating System]
toc: true
date: 2022-01-21 14:40
updated: 2022-01-21 14:40
references:
  - title: 八股文（星球精华汇总）
    url: https://wx.zsxq.com/dweb2/index/topic_detail/185515824245112
  - title: 理解文件系统 - 掘金
    url: https://juejin.cn/post/6916446785775468557
---

文件系统是操作系统中负责管理持久数据的子系统，即负责把用户的文件存到磁盘硬件中。

因为即使计算机断电了，磁盘里的数据并不会丢失，所以可以持久化的保存文件。

文件是对长期存储介质的抽象。

<!-- more -->

## 文件

文件是⼀种抽象机制，它提供了⼀种在磁上保存信息而且方便以后读取的方法。
这种方法可以使用户不必了解存储信息的方法、位置和实际磁盘⼯作方式等有关细节。

> win95、win98 用的都是 MS-DOS 的文件系统，即 FAT-16， win98 扩展了 FAT-16 成为 FAT-32。
> 较新版的操作系统 NTFS，win8 配备 ReFS。微软优化 FAT,叫作 exFAT。
> prog.c，圆点后面的部分称为文件扩展名。

### 文件结构

- 字节结构：把文件看成字节序列为操作系统提供了最大的灵活度
- 记录序列：文件结构上的第⼀步改进，这种模型中，文件是具有固定长度记录的序列
- 树：文件在这种结构中由⼀棵记录树构成，每个记录不必具有相同的长度，记录的固定位置上有⼀个键字段。这棵树按“键”字段进行排序，从而可以对特定“键”进行快速查找。

### 文件类型

1. 普通文件
2. 目录
3. 字符特殊文件（UNIX）
4. 块特殊文件（UNIX）

### 文件属性

除了文件名和数据外，文件还具有**属性**来对文件本身做更具体的描述，这类信息也称为**元数据**。
这些属性会存于文件结构中的某些区域中，具体要视该文件的类型而定。

**举例**：

- 创建时间、修改时间、存取时间、文件大小、当前大小、所有者等
- 保护：对文件的访问限制，谁可以访问文件
- 口令：访问文件需要的密码
- 只读标志、隐藏标志、系统标志（普通文件或系统文件）、加锁标志、存档标志
- 最大长度：文件可能增长到的字节数

### 文件操作

使用文件的目的是**存储信息并方便以后检索**。对于存储和检索，不同系统提供了不同的操作。

文件系统大多都会提供如下的**文件操作**：

- create()：创建不包含任何数据的文件。
- delte()：删除该文件以释放磁盘空间。
- open()：在使用文件之前，先打开文件，目的是把文件属性和磁盘地址表装入内存，以便后续调用的快速访问。
- close()：文件访问结束之后，关闭文件并释放内部空间表空间。
- write()：向文件写数据，如果当前位置是文件尾，那么数据长度会增加；如果当前位置是其中某个位置，那么写入位置的数据将会被覆盖。
- append()：相当于 write() 在尾部添加数据。
- seek()：随机访问文件，需要指定相对于文件数据开始的位置。
- getAttribute()：读取文件的属性。
- setAttribute()：设置文件的属性。
- rename()：重命名文件。

一般来说，打开文件将会获得一个**文件描述符**(一个小整数)，通过文件描述符，就可以操作文件。

## 目录

文件系统通常提供目录或文件夹用于记录文件的位置，在很多操作系统中目录本身也是文件

现在的文件目录结构，大多采用文件树结构。
这里的路径，用**路径名**表示，分为**绝对路径名**和**相对路径名**，路径每往下一层，使用分隔符进行分隔。

分隔符视系统而定，如下：

```
> Windows：\usr\ast\maibox
> Linux: /usr/ast/mailbox
> MULTICS: >usr>ast>mailbox
```

### 目录操作

对于目录，也提供了相应的操作：

- create(): 创建目录
- delete(): 删除目录
- opendir(): 打开目录
- closedir(): 关闭目录，释放内部表空间
- readdir(): 返回一个目录项，在内存中，以目录项来表达一个目录
- rename(): 重命名
- link(): 链接一个文件，之后能使同一个文件，能通过多个文件路径访问到 (后续会提到)
- unlink(): 解除文件的链接

## 文件系统

以上内容是以用户（即使用者）的角度看待文件。
那么，从文件系统实现者的角度来看，文件系统应该如何实现？

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202201232026680.png" style="zoom:67%;" />

文件系统存放在磁盘上，磁盘被划分为一个或者多个分区，每个分区中有一个独立的文件系统。
磁盘的 0 号扇区称为**主引导记录（Master Boot Record, MBR）**，用来引导计算机。
紧接着，是分区表，用来标记每个分区的起始和结束位置，表中的一个分区被标记为活动分区。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202201232028987.png" style="zoom:50%;" />

> 当计算机被引导时，BIOS 读入并执行 MBR。
> MBR 会确定活动分区，读入并执行它的**引导块（boot block）**。引导块中的程序将装载该分区中的系统。
> **超级块**（superblock）包含文件系统的所有关键参数，会被读入内存，其中的信用可用来确定文件系统的魔数、文件系统中的块的数量和其他重要的管理信息。
>
> 空闲块的信息可以用位图或者指针的形式指出，随后跟随的是一组 i 节点（一种数据结构，用来说明文件的方方面面）。最后，存放了所有其他的文件和目录。

### 文件的实现

#### 连续分配

最简单的分配是把每个文件作为⼀连串连续数据块存储在磁盘

**优点：**

- 实现简单，只需记住开始位置，文件的块数。
- 读操作性能好，单个操作就能从磁盘中读出整个文件。只需要一次寻找，之后不再有寻道和旋转延迟。

**缺点：**

- 尾部会浪费一些空间，以块为存储单位时，无论是否完全使用完块大小，都会占用整个块
- 需要知道文件的最终大小，然后，在维护的连续空闲表找到合适的位置存入文件，并且知道文件的最终大小这一问题不可回避。
- 随时间迁移，将会产生大量碎片块，使磁盘变得零碎，当一个较大的新文件要加入时，将找不到合适的连续位置，此时要压缩磁盘空间（把文件复制到新位置，以得到更多的连续空间），代价极高。

> 新建文件时要知道文件的最终大小，使得这一存储方式难以应用于实际场景，但是在一些情形下是可行的。
>
> 如在 CD-ROM、DVD、BD（蓝光光盘）上，在它们的场景里，所有的文件的大小是事先知道的，并且在后续的使用中，这些文件的大小也不会改变。

#### 链表分配

可以为每个文件构造磁盘块链表，每一块的第一个字指向下块，块的其他部分存放数据。

**优点：**

- 不会因为磁盘碎片浪费存储空间，充分利用磁盘块。
- 只需要第一块的磁盘地址，就能找到其他的块。

**缺点：**

- 随机方法访问很慢，访问块 n 总是要先访问其面的 n-1 块。
- 指向下一块的指针占据了一些字节，每个磁盘块存储的字节数不再是 2 的整次幂，因大多数程序以长度为 2 的整次幂来读写磁盘块，降低了实际系统的运行效率。
- 要读出一个完成的文件块时，要从两个磁盘块中获得和拼接信息，带来额外的开销

#### 内存中的表进行链表分配

弥补链表分配的不足，可以在内存中建立磁盘块的指针表。

表项中，使用一个特殊的标记（如 -1）表示结束。这样，沿着表就可以文件的所有块。

这样的表也称为 **FAT（File Allocation Table）** 。

**优点：**

- 整个块都可以存放数据。
- 随机访问也简单得多，虽然仍要顺着链表寻找偏移量（但减少了可能的寻到和旋转延迟）
- 不需要磁盘引用

**缺点：**

- 必须把整个表都放在内存中。假设 1TB 的磁盘和 1KB 的块大小，则表需要 10 亿项，每一项至少要 3 个字节，则表大小要占用 2.4GB 内存。因此 FAT 的应用场景有限，不太实用。

#### I 节点

最普遍的方式，是为每个文件创建一个中数据结构以表示此文件的关键描述，这种数据结构也称为**i 节点（index-node）**。

一个可能的 inode 结构，记录了文件属性，以及每个文件块对应的磁盘块地址，并能指向其他的 inode 来记录更多的存储数据。

这样的机制有很大的优势：

- 给定 inode，就能找到所有的文件块。
- 只有在文件打开时，inode 才会存在于内存中，如果每个 i 节点有 n 字节，那么 k 个文件同时打开，inode 总共占据 kn 字节，只需提前保留这么多的空间即可。也就是说，内存中 inode 占据的大小和文件大小无关，只和同时使用的文件数有关。
- 如果一个 inode 不够存储一个文件占用的所有磁盘块，可以通过最后一个地址之前其他 inode 得以解决。甚至可以指向其他存放地址的磁盘的磁盘块。

### 目录的实现

在读文件之前，必须先打开文件。打开文件时，操作系统利用用户给出的路径名找到相应的目录项。

- 简单目录：包含固定大⼩的目录，在目录项中有磁盘地址和属性
- 采用 i 节点的系统：把文件属性存放在 i 节点中而不是目录项中。这种情形下，目录项会更短。

### 共享文件

共享文件与目录的联系称为⼀个链接（link）。
这样文件系统本身就是⼀个有向⽆环图（DAG），而不是⼀棵树。

- 硬链接：指向目标数据对象的链接，**可以看作是一个既有文件的别名**

> 当目标被删除时，硬链接继续存在，且可以正常打开、编辑。因为他具备一个完整的文件结构。
>
> 当硬链接被删除时，目标文件继续存在，不受影响。
>
> 只有当一个文件 ID 对应的所有硬链接被删除时，数据才真正被标记为删除。

- 符号链接：指向目标路径的链接，会跳转到符号链接所指向的目标中去，而不改变此时的文件路径

> 当目标被删除时，符号链接继续存在，但会成为死链，无法打开。
>
> 当符号链接被删除时，它指向的目标不受影响。

### 虚拟文件系统

将多个文件系统整合到⼀个统⼀的结构中。

> ⼀个 Linux 系统可以用 `ext2` 作为根文件系统，`ext3` 分区装载在 `/usr` 下，另⼀块采用 ReiserFS 文件系统的硬盘装载在 `/home` 下，以及⼀个 ISO 9660 的 CD-ROM 临时装载在 `/mnt` 下。

从用户的观点来看，只有⼀个文件系统层级。但它们事实上是多种文件系统，对于用户和进程是不可见的。

绝大多数 Unix 操作系统都在使用虚拟文件系统（VirtualFile System, VFS）

### 文件系统面对的问题

#### 磁盘空间管理

几乎所有的文件系统将文件分割成固定大小的块来存储，各块之间不一定相邻。

##### 块大小

块大小将直接影响此消彼长的两个指标**空间利用率**和**磁盘数据率**。

> 从历史的观点上来说，文件系统将大⼩设在 1~4KB 之间。
>
> 但现在随着磁盘超过了 1TB，还是将块的大⼩提升到 64KB 并且接受浪费的磁盘孔空间，这样也许更好。磁盘空间⼏乎不再会短缺。

##### 空闲块记录

确定了块大小，需要考虑空闲块如何记录。

- 链表记录：链表的每个块中包含尽可能多的空闲磁盘块号。（通常情况下，采用空闲块存放空闲表，这样不会影响存储器）
- 位图记录：在位图中，空闲块用 1 表示，已分配块用 0 表示。

##### 文件备份

做磁盘备份主要是处理好两个潜在问题中的⼀个：

1. 从意外的灾难中恢复
2. 从错误的操作中恢复

将文件从磁盘转储到磁带，有两种方法，**物理转储**和**逻辑转储**。

#### 文件系统的一致性

文件进行修改后，需将所有的修改都写回磁盘，如果在所有的磁盘块都写回磁盘前，系统崩溃，一些被修改的信息可能未被写回磁盘，那么文件系统将不一致。

因此，文件系统一般都有独立的程序可检查其一致性。

致性检查分为两种**块的一致性检查**和**文件的一致性检查**

#### 文件系统的性能

磁盘访问的速度比内存访问的速度慢得多，如果每次读文件时，都从磁盘块中读取，那么 IO 时间将是大大拖累程序效率。

主要通过以下方法提高性能：

- 高速缓存：减少磁盘访问次数技术是块高速缓存（block cache）或者缓冲区高速缓存（buffer cache）
- 块预读：在需要用到块之前，试图提前将其写入高速缓存，从而提高命中率。
- 减少磁盘臂运动：把可能顺序访问的块放⼀起，当然最好是同⼀柱面上，从而减少磁盘臂的移动次数。
- 磁盘碎片整理：移动文件使它们相邻，并把所有的空闲空间放在⼀个或多个大的连续区域内。
