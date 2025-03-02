---
title: KMP 算法
date: 2021-10-19 19:56
tags:
  - String
categories: [Computer Basics,Algorithm]
toc: true
updated: 2021-10-19 19:56
---

KMP 主要应用在字符串匹配上。

KMP 的主要思想是**当出现字符串不匹配时，可以知道一部分之前已经匹配的文本内容，可以利用这些信息避免从头再去做匹配了。**

<!-- more -->

## 说明

借用了 [这篇博客](https://www.cnblogs.com/dusf/p/kmp.html) 的一些插图和理解

next 数组就是一个前缀表（prefix table）：**前缀表是用来回退的，它记录了模式串与主串 (文本串) 不匹配的时候，模式串应该从哪里开始重新匹配。**

假设 匹配串 S， 模式串 T

![初始状态](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2Jsb2cvNDE2MDEwLzIwMTMwOC8xNzA4MzkxMi00OTM2NWI3ZTY3Y2Q0ODc3YjJmNTAxMDc0ZGFlNjhkMi5wbmc?x-oss-process=image/format,png)

- **最长公共前后缀** ：
	ABCAB 公共前后缀 是 AB
	ABABA 公共前后缀 是 ABA
	ABCABC 公共前后缀 是 ABC

如图，S[3]!=T[3]（i=3，j=3），但前面的 ABA 有公共前后缀 A (最长公共前后缀不能等于前面的串长)，长度为 1，所以把 j 移到 1（前缀移到后缀处）， 如下图

![不匹配](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2Jsb2cvNDE2MDEwLzIwMTMwOC8xNzA4MzkyOS1hOWNjZmIwODgzM2U0Y2YxYTQyYzMwZjA1NjA4ZjhmNS5wbmc?x-oss-process=image/format,png)

又比如 下图中，S[5]!=T[5]（i=5，j=5），有公共前后缀 AB，长度 2

将 j 移到 2 （i=5，j=2）

<table>
<tr>
<th>
<img src="https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2Jsb2cvNDE2MDEwLzIwMTMwOC8xNzA4NDAzMC04MmU0YjcxYjg1YTQ0MGM1YTYzNmQ1NzUwMzkzMTQxNS5wbmc?x-oss-process=image/format,png">
</th>
<th>
<img src="https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2Jsb2cvNDE2MDEwLzIwMTMwOC8xNzA4NDAzNy1jYzNjMzQyMDA4MDk0MTRlOTQyMWMzMTZjZWJhMmNkYS5wbmc?x-oss-process=image/format,png">
</tr>
</table>

所以，重点要求 next 数组：

**next[i] : 满足 x[i-k…i-1]=x[0…k-1] 的最大 k 值**
（最长公共前后缀的长度）

考虑四种情况：

- ① j = 0 ， 如果不匹配，j 已经在最左边，无法回退，所以要右移，即初始化 next [ 0 ] = -1

![j=0](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2Jsb2cvNDE2MDEwLzIwMTMwOC8xNzA4NDI1OC1lZmQyZTk1ZDM2NDQ0MjdlYmMwMzA0ZWQzZDdhZGVmYi5wbmc?x-oss-process=image/format,png)

- ② j = 1 , 如果不匹配，只能左移到 0 ，所以 next [ 1 ] = 0
	![j=1](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2Jsb2cvNDE2MDEwLzIwMTMwOC8xNzA4NDMxMC0yOWY5ZjhkYmI2MDM0MTUxYTM4M2U3Y2NmNmY1NTgzZS5wbmc?x-oss-process=image/format,png)
- ③ X [ k ] = X [ j ]
	匹配到这个位置说明失配位置前 公共前后缀相等，即
	X [ 0…k-1 ] = X [ j-k…j-1] （k 为公共前后缀长度）
	那么，X [ 0…k ] = X [ j-k…j ]
	所以，next [ j+1 ] = k+1 (数组下标从 0 开始，长度为 k+1)
	![匹配](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2Jsb2cvNDE2MDEwLzIwMTMwOC8xNzA4NDMyNy04YTNjZGZhYjAzMDk0YmZhOWU1Y2FjZTI2Nzk2Y2FlNS5wbmc?x-oss-process=image/format,png)
- ④ X [ k ] != X [ j ]
	next[i] : 满足 x[j-k…i-1]=x[0…j-1] 的最大 k 值 (k 为最长公共前后缀)
	此时，**将前缀移到后缀位置**，即指针前移到最长公共前后缀的长度位置，可以得出：
	k = next [ k ]
	![不匹配](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2Jsb2cvNDE2MDEwLzIwMTMwOC8xNzEyMjM1OC1mZDdlNTJkZDM4MmM0MjY4YThmZjUyYjg1YmZmNDY1ZC5wbmc?x-oss-process=image/format,png)

不过，这样求得的 next 数组还有缺陷：

![缺陷](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2Jsb2cvNDE2MDEwLzIwMTMwOC8xNzA4NDcxMi1mMGQ2OTk4OTM4NzY0YjMwOWY2MTkyMzQ1MmEyYjIwZi5wbmc?x-oss-process=image/format,png)

按照前述，j 移到 最长公共前后缀长度 1 的位置：

这一步是完全没有意义的。因为后面的 B 已经不匹配了，那前面的 B 也一定是不匹配的，即 **X[ j ] = X[ next[ j ] ] 的情况没有意义**

![重复](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2Jsb2cvNDE2MDEwLzIwMTMwOC8xNzA4NDcyNi03OTBmYzFiMmM0OGM0MTFiODAxMWVhYjlkZTY5MmY2ZC5wbmc?x-oss-process=image/format,png)

<a href="https://github.com/youngyangyang04/leetcode-master/blob/master/problems/0028.%E5%AE%9E%E7%8E%B0strStr.md">代码随想录动图示例：</a>

<img src="https://camo.githubusercontent.com/49f9392ca7baccbfbaec6823dbad095a6a582a7d1a101b4f37e2fa3eb5003061/68747470733a2f2f636f64652d7468696e6b696e672e63646e2e626365626f732e636f6d2f676966732f4b4d50254537254232254245254538254145254232332e676966">

## 代码

### GO

```go
func getNext(next []int, s string) {
	i, j := 0, -1
	next[0] = j
	for i < len(s) {
		for j !=-1 && s[i] != s[j] { // 前后缀不同，向前回退
			j = next[j]
		}
		i++
		j++
		if i >= len(s) || j >= len(s) {
			return
		}
		if s[i] == s[j] { // 优化去重
			next[i] = next[j]
		} else {
			next[i] = j
		}
	}
}

func strStr(haystack string, needle string) int {
	if len(needle) == 0 {
		return 0
	}
	next := make([]int, len(needle))
	getNext(next, needle)
	i, j := 0, 0
	for i < len(haystack) && j < len(needle) {
		for j !=-1 && haystack[i] != needle[j] {
			j = next[j]
		}
		i++
		j++
	}
	if j == len(needle) {
		return i - j
	} else {
		return -1
	}
}
```

### C++

```c++
int nx[maxn];
void getnx(string x){
	int m=x.size();
	int i,j;
	j=nx[0]=-1;
	i=0;
	while (i<m){
		while (-1!=j && x[i]!=x[j]) j=nx[j];
		if (x[++i]==x[++j]) nx[i]=nx[j];
		else nx[i]=j;
	}
}
//输出第一次出现的位置
int kmp(string s, string t) { //s为匹配串，t为模式串
	int n=s.size();
	int m=t.size();
	int i=0,j=0;
	getnx(t);
	while (i<n && j<m) {
		while (-1!=j && s[i]!=t[j]) j=nx[j];
		i++;
		j++;
	}
	if (j==m)
		return i-j;
	else return -1;
}
```

## 性质

**数组长度减去最长相同前后缀的长度相当于是第一个周期的长度，也就是一个周期的长度，如果这个周期可以被整除，就说明整个数组就是这个周期的循环。**

即**最小的循环节**

```go
func getNext(next []int, s string) { // 计算后缀，不需要去重
	i, j := 0, -1
	next[0] = j
	for i < len(s) {
		for j != -1 && s[i] != s[j] { // 前后缀不同，向前回退
			j = next[j]
		}
		i++
		j++
		next[i] = j
	}
}

func repeatedSubstringPattern(s string) bool { // 
	if len(s) == 0 {
		return false
	}
	next := make([]int, len(s)+1) // 多+1 计算末位最长前后缀
	getNext(next, s)
	l := len(s)
	if next[l] != 0 && l%(l-next[l]) == 0 {
		return true
	}
	return false
}
```
