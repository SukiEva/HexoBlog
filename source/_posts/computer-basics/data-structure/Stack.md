---
title: 栈 Stack
date: 2021-10-28 14:32
tags:
  - Stack
categories: [Computer Basics,Data Structure]
toc: true
updated: 2021-10-28 14:32
references:
  - title: 队列 & 栈 - LeetBook - 力扣（LeetCode）
    url: https://leetcode-cn.com/leetbook/read/queue-stack/gxtls/
  - title: 代码随想录 (programmercarl.com)
    url: https://programmercarl.com/栈与队列理论基础.html
---

栈是以底层容器完成其所有的工作，对外提供统一的接口，底层容器是可插拔的（即可以控制使用哪种容器来实现栈的功能）。

STL 中栈往往不被归类为容器，而被归类为 container adapter（容器适配器）。

<!-- more -->

## 介绍

在 LIFO 数据结构中，将首先处理添加到队列中的最新元素。

与队列不同，栈是一个 LIFO 数据结构。通常，插入操作在栈中被称作入栈 push 。与队列类似，总是在堆栈的末尾添加一个新元素。但是，删除操作，退栈 pop ，将始终删除队列中相对于它的最后一个元素。

<img src="https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/06/03/screen-shot-2018-06-02-at-203523.png" alt=" 后入先出的数据结构 " style="zoom: 50%;" />

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/%E5%87%BA%E5%85%A5%E6%A0%88.gif" alt=" 入栈与出栈 " style="zoom:50%;" />

## 实现

从下图中可以看出，栈的内部结构，栈的底层实现可以是 vector，deque，list 都是可以的， 主要就是数组和链表的底层实现。

<img src="https://img-blog.csdnimg.cn/20210104235459376.png" alt=" 栈与队列理论 3" style="zoom:50%;" />

**常用的 SGI STL，如果没有指定底层实现的话，默认是以 deque 为缺省情况下栈的低层结构。**

deque 是一个双向队列，只要封住一段，只开通另一端就可以实现栈的逻辑了。

**SGI STL 中 队列底层实现缺省情况下一样使用 deque 实现的。**

```c++
std::stack<int, std::vector<int> > third;  // 使用vector为底层容器的栈
std::queue<int, std::list<int>> third; // 定义以list为底层容器的队列
```

**动态数组实现参考：**

```c++
#include <iostream>

class MyStack {
    private:
        vector<int> data;               // store elements
    public:
        /** Insert an element into the stack. */
        void push(int x) {
            data.push_back(x);
        }
        /** Checks whether the queue is empty or not. */
        bool isEmpty() {
            return data.empty();
        }
        /** Get the top item from the queue. */
        int top() {
            return data.back();
        }
        /** Delete an element from the queue. Return true if the operation is successful. */
        bool pop() {
            if (isEmpty()) {
                return false;
            }
            data.pop_back();
            return true;
        }
};

int main() {
    MyStack s;
    s.push(1);
    s.push(2);
    s.push(3);
    for (int i = 0; i < 4; ++i) {
        if (!s.isEmpty()) {
            cout << s.top() << endl;
        }
        cout << (s.pop() ? "true" : "false") << endl;
    }
}
```
