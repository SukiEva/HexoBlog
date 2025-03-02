---
title: 二叉树遍历
date: 2021-11-03 08:47
tags:
  - Tree
categories: [Computer Basics,Algorithm]
toc: true
updated: 2021-11-03 08:47
references:
  - title: 代码随想录 (programmercarl.com)
    url: https://programmercarl.com/二叉树的递归遍历.html
---

二叉树遍历主要包括：

- 深度优先遍历
	- 前序遍历（递归法，迭代法）
	- 中序遍历（递归法，迭代法）
	- 后序遍历（递归法，迭代法）
- 广度优先遍历
	- 层次遍历（迭代法）

<!-- more -->

## 递归遍历

**递归三要素：**

1. **确定递归函数的参数和返回值：** 确定哪些参数是递归的过程中需要处理的，那么就在递归函数里加上这个参数， 并且还要明确每次递归的返回值是什么进而确定递归函数的返回类型。
2. **确定终止条件：** 写完了递归算法, 运行的时候，经常会遇到栈溢出的错误，就是没写终止条件或者终止条件写的不对，操作系统也是用一个栈的结构来保存每一层递归的信息，如果递归没有终止，操作系统的内存栈必然就会溢出。
3. **确定单层递归的逻辑：** 确定每一层递归需要处理的信息。在这里也就会重复调用自己来实现递归的过程。

### 前序遍历

即 根 - 左 - 右

```go
func preorderTraversal(root *TreeNode) []int {
	ans := make([]int, 0)
	var preOrder func(root *TreeNode)
	preOrder = func(root *TreeNode) {
		if root == nil {
			return
		}
		ans = append(ans, root.Val)
		preOrder(root.Left)
		preOrder(root.Right)
	}
	preOrder(root)
	return ans
}
```

### 中序遍历

即 左 - 根 - 右

```go
func inorderTraversal(root *TreeNode) []int {
	ans := make([]int, 0)
	var inOrder func(root *TreeNode)
	inOrder = func(root *TreeNode) {
		if root == nil {
			return
		}
		inOrder(root.Left)
		ans = append(ans, root.Val)
		inOrder(root.Right)
	}
	inOrder(root)
	return ans
}
```

### 后序遍历

即 左 - 右 - 根

```go
func postorderTraversal(root *TreeNode) []int {
	ans := make([]int, 0)
	var postorder func(root *TreeNode)
	postorder = func(root *TreeNode) {
		if root == nil {
			return
		}
		postorder(root.Left)
		postorder(root.Right)
		ans = append(ans, root.Val)
	}
	postorder(root)
	return ans
}
```

## 迭代遍历

**要处理的节点放入栈之后，紧接着放入一个空指针作为标记。** 通过这种标记法实现二叉树的统一迭代遍历。

![中序遍历](https://tva1.sinaimg.cn/large/008eGmZEly1gnbmq3btubg30em09ue82.gif)

### 前序遍历

```go
func preorderTraversal(root *TreeNode) []int {
	if root == nil {
		return nil
	}
	ans := make([]int, 0)
	stack := list.New()
	stack.PushBack(root)
	var node *TreeNode
	for stack.Len() > 0 {
		back := stack.Back()
		stack.Remove(back)
		if back.Value != nil { 
			node = back.Value.(*TreeNode) // interface 为 nil 无法 断言类型转换
			if node.Right != nil { 
				stack.PushBack(node.Right) // 右
			}
			if node.Left != nil {
				stack.PushBack(node.Left) // 左
			}
			stack.PushBack(node) // 根
			stack.PushBack(nil)
		} else {
			node = stack.Remove(stack.Back()).(*TreeNode)
			ans = append(ans, node.Val)
		}
	}
	return ans
}
```

### 中序遍历

```go
func inorderTraversal(root *TreeNode) []int {
	if root == nil {
		return nil
	}
	ans := make([]int, 0)
	stack := list.New()
	stack.PushBack(root)
	var node *TreeNode
	for stack.Len() > 0 {
		back := stack.Back()
		stack.Remove(back)
		if back.Value != nil {
			node = back.Value.(*TreeNode) // interface 为 nil 无法 断言类型转换
			if node.Right != nil {
				stack.PushBack(node.Right) // 右
			}
			stack.PushBack(node) // 根
			stack.PushBack(nil)
			if node.Left != nil {
				stack.PushBack(node.Left) // 左
			}
		} else {
			node = stack.Remove(stack.Back()).(*TreeNode)
			ans = append(ans, node.Val)
		}
	}
	return ans
}
```

### 后序遍历

```go
func postorderTraversal(root *TreeNode) []int {
	if root == nil {
		return nil
	}
	ans := make([]int, 0)
	stack := list.New()
	stack.PushBack(root)
	var node *TreeNode
	for stack.Len() > 0 {
		back := stack.Back()
		stack.Remove(back)
		if back.Value != nil {
			node = back.Value.(*TreeNode) // interface 为 nil 无法 断言类型转换
			stack.PushBack(node) // 根
			stack.PushBack(nil)
			if node.Right != nil {
				stack.PushBack(node.Right) // 右
			}
			if node.Left != nil {
				stack.PushBack(node.Left) // 左
			}
		} else {
			node = stack.Remove(stack.Back()).(*TreeNode)
			ans = append(ans, node.Val)
		}
	}
	return ans
}
```

## 层序遍历

```go
func levelOrder(root *TreeNode) [][]int {
	ans := [][]int{}
	if root == nil {
		return ans
	}
	queue := list.New()
	queue.PushBack(root)
	t := make([]int, 0)
	for queue.Len() > 0 {
		l := queue.Len()
		for i := 0; i < l; i++ {
			node := queue.Remove(queue.Front()).(*TreeNode)
			t = append(t, node.Val)
			if node.Left != nil {
				queue.PushBack(node.Left)
			}
			if node.Right != nil {
				queue.PushBack(node.Right)
			}
		}
		ans = append(ans, t)
		t = []int{}
	}
	return ans
}
```
