---
title: 队列 Queue
date: 2021-10-28 15:00
tags:
  - Queue
categories: [Computer Basics,Data Structure]
toc: true
updated: 2021-10-28 15:00
references:
  - title: 队列 & 栈 - LeetBook - 力扣（LeetCode）
    url: https://leetcode-cn.com/leetbook/read/queue-stack/k6zxm/
---

与栈类似，队列是一种**先进先出**的容器适配器。

<!-- more -->

## 定义

<img src="https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/08/14/screen-shot-2018-05-03-at-151021.png" alt=" 先入先出的数据结构 " style="zoom:50%;" />

在 FIFO 数据结构中，将首先处理添加到队列中的第一个元素。

如上图所示，队列是典型的 FIFO 数据结构。插入（insert）操作也称作入队（enqueue），新元素始终被添加在队列的末尾。 删除（delete）操作也被称为出队（dequeue)。 你只能移除第一个元素。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/%E5%87%BA%E5%85%A5%E9%98%9F.gif" alt=" 入队与出队 " style="zoom:50%;" />

## 实现

为了实现队列，我们可以使用动态数组和指向队列头部的索引。

**简单的参考实现：**

```c++
#include <iostream>

class MyQueue {
    private:
        // store elements
        vector<int> data;       
        // a pointer to indicate the start position
        int p_start;            
    public:
        MyQueue() {p_start = 0;}
        /** Insert an element into the queue. Return true if the operation is successful. */
        bool enQueue(int x) {
            data.push_back(x);
            return true;
        }
        /** Delete an element from the queue. Return true if the operation is successful. */
        bool deQueue() {
            if (isEmpty()) {
                return false;
            }
            p_start++;
            return true;
        };
        /** Get the front item from the queue. */
        int Front() {
            return data[p_start];
        };
        /** Checks whether the queue is empty or not. */
        bool isEmpty()  {
            return p_start >= data.size();
        }
};

int main() {
    MyQueue q;
    q.enQueue(5);
    q.enQueue(3);
    if (!q.isEmpty()) {
        cout << q.Front() << endl;
    }
    q.deQueue();
    if (!q.isEmpty()) {
        cout << q.Front() << endl;
    }
    q.deQueue();
    if (!q.isEmpty()) {
        cout << q.Front() << endl;
    }
}
```

### 缺点

上面的实现很简单，但在某些情况下效率很低。 随着起始指针的移动，浪费了越来越多的空间。 当我们有空间限制时，这将是难以接受的。

所以应该规定数组的长度，一组一组队列的进行使用。
