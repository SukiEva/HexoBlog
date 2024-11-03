---
title: Binary Search 二分查找
toc: true
date: 2021-11-21 16:23
tags:
  - Search
categories: [Computer Basics,Algorithm]
updated: 2021-11-21 16:23
---

二分查找也称折半查找，它是一种效率较高的查找方法。

<!-- more -->

## 前提

数组为有序数组，数组中无重复元素。

因为一旦有重复元素，使用二分查找法返回的元素下标可能不是唯一的，这些都是使用二分法的前提条件。

二分查找涉及的很多的边界条件，逻辑比较简单，但就是写不好。

- 到底是 while(left < right) 还是 while(left <= right)？
- 到底是 right = middle 呢，还是要 right = middle - 1 呢？

区间的定义就是不变量。要在二分查找的过程中，保持不变量，就是在 while 寻找中每一次边界的处理都要坚持根据区间的定义来操作，这就是循环不变量规则。

写二分法，区间的定义一般为两种，左闭右闭即 [left, right]，或者左闭右开即 [left, right)。

## 写法

### 左闭右闭

定义 target 是在一个在左闭右闭的区间里，也就是 [left, right]

区间的定义这就决定了二分法的代码应该如何写，因为定义 target 在 [left, right] 区间，所以有如下两点：

- while (left <= right) 要使用 <= ，因为 left == right 是有意义的，所以使用 <=
- if (nums[middle] > target) right 要赋值为 middle - 1，因为当前这个 nums[middle] 一定不是 target，那么接下来要查找的左区间结束下标位置就是 middle - 1

**代码：**

```c++
int search(vector<int>& nums, int target) {  // 左闭右闭
    int left = 0, right = nums.size() - 1;
    while (left <= right) {
        int middle = left + ((right - left) >> 1);  // 防止溢出
        if (nums[middle] > target)
            right = middle - 1;
        else if (nums[middle] < target)
            left = middle + 1;
        else
            return middle;
    }
    return -1;
}
```

**Go**

```go
func search(nums []int, target int) int {
	left, right := 0, len(nums)-1
	for left <= right {
		middle := left + (right-left)>>1
		if nums[middle] > target {
			right = middle - 1
		} else if nums[middle] < target {
			left = middle + 1
		} else {
			return middle
		}
	}
	return -1
}
```

### 左闭右开

定义 target 是在一个在左闭右开的区间里，也就是 [left, right) ，那么二分法的边界处理方式则截然不同。

有如下两点：

- while (left < right)，这里使用 < ,因为 left == right 在区间 [left, right) 是没有意义的
- if (nums[middle] > target) right 更新为 middle，因为当前 nums[middle] 不等于 target，去左区间继续寻找，而寻找区间是左闭右开区间，所以 right 更新为 middle，即：下一个查询区间不会去比较 nums[middle]

**代码：**

```c++
int search(vector<int>& nums, int target) {
    int left = 0;
    int right = nums.size();  // 左闭右开
    while (left < right) {    // left == right在[left, right)是无效的空间
        int middle = left + ((right - left) >> 1);
        if (nums[middle] > target) {
            right = middle;  // target 在左区间，在[left, middle)中
        } else if (nums[middle] < target) {
            left = middle + 1;  // target 在右区间，在[middle + 1, right)中
        } else {                // nums[middle] == target
            return middle;      // 数组中找到目标值，直接返回下标
        }
    }
    // 未找到目标值
    return -1;
}
```

## 进阶

### 第一个大于/等于

在二分过程中，更新答案。

如果找第后一个小于也类似。

```c++
int bsh(vector<int>& nums, int target, bool flag) { // flag true：第一个大于等于
	int l = 0, r = nums.size() - 1, ans = nums.size();
    while (l <= r) {
    	int m = l + ((r - l) >> 1);
        if (nums[m] > target || (flag && nums[m] >= target)) {
        	r = m - 1;
            ans = m;
        } else l = m + 1;
     }
     return ans;
}
```
