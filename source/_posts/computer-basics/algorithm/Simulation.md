---
title: 模拟题
tags:
  - Simulation
categories: [Computer Basics,Algorithm]
toc: true
date: 2022-02-18 10:33
updated: 2022-02-18 10:33
references:
  - title: 【宫水三叶】使用「双栈」解决「究极表达式计算」问题
    url: https://leetcode-cn.com/problems/basic-calculator-ii/solution/shi-yong-shuang-zhan-jie-jue-jiu-ji-biao-c65k/
---

一些经典模拟题

<!-- more -->

## 计算器

对于「任何表达式」而言，都使用两个栈 `nums` 和 `ops`：

- `nums` ： 存放所有的数字
- `ops` ：存放所有的数字以外的操作

从前往后遍历字符，分情况讨论：

- 空格 : 跳过（开始去除所有空格）
- `(` : 直接加入 `ops` 中，等待与之匹配的 `)`
- `)` : 使用现有的 `nums` 和 `ops` 进行计算，直到遇到左边最近的一个左括号为止，计算结果放到 `nums`
- 数字 : 从当前位置开始继续往后取，将整一个连续数字整体取出，加入 `nums`
- `+ - * / ^ %` : 需要将操作放入 `ops` 中。**在放入之前先把栈内可以算的都算掉（只有「栈内运算符」比「当前运算符」优先级高/同等，才进行运算）**，使用现有的 `nums` 和 `ops` 进行计算，直到没有操作或者遇到左括号，计算结果放到 `nums`

**只有「栈内运算符」比「当前运算符」优先级高/同等，才进行运算** 是什么意思：
从前往后做，假设我们当前已经扫描到 `2 + 1` 了（此时栈内的操作为 `+` ）。

1. 如果后面出现的 `+ 2` 或者 `- 1` 的话，满足「栈内运算符」比「当前运算符」优先级高/同等，可以将 `2 + 1` 算掉，把结果放到 `nums` 中；
2. 如果后面出现的是 `* 2` 或者 `/ 1` 的话，不满足「栈内运算符」比「当前运算符」优先级高/同等，这时候不能计算 `2 + 1`。

> - 由于第一个数可能是负数，为了减少边界判断。一个小技巧是先往 `nums` 添加一个 0
> - 从理论上分析，`nums` 最好存放的是 `long`，而不是 `int`，可能存在 中间结果溢出

```c++
typedef long long ll;

class Solution {
   private:
    unordered_map<char, int> opMap = {{'+', 1}, {'-', 1}, {'*', 2},
                                      {'/', 2}, {'%', 2}, {'^', 3}};
    stack<ll> nums;
    stack<char> ops;
    void calc() {
        if (nums.size() < 2 || ops.empty()) return;
        ll b = nums.top();
        nums.pop();
        ll a = nums.top();
        nums.pop();
        char op = ops.top();
        ops.pop();
        ll ans;
        switch (op) {
            case '+':
                ans = a + b;
                break;
            case '-':
                ans = a - b;
                break;
            case '*':
                ans = a * b;
                break;
            case '/':
                ans = a / b;
                break;
            case '%':
                ans = a % b;
                break;
            case '^':
                ans = pow(a, b);
                break;
        }
        nums.push(ans);
    }

   public:
    int calculate(string s) {
        // 去除所有空格
        s.erase(remove(s.begin(), s.end(), ' '), s.end());
        nums.push(0);  // 防止第一个数是负数
        int n = s.size();
        for (int i = 0; i < n; i++) {
            char c = s[i];
            if (c == '(')
                ops.push(c);
            else if (c == ')') {  // 计算到最近的左括号
                while (!ops.empty()) {
                    if (ops.top() == '(') {
                        ops.pop();
                        break;
                    }
                    calc();
                }
            } else {
                if (c >= '0' && c <= '9') {
                    int num = 0, j;
                    for (j = i; j < n && s[j] >= '0' && s[j] <= '9'; j++)
                        num = num * 10 + (s[j] - '0');
                    nums.push(num);
                    i = j - 1;
                } else {
                    if (i > 0 &&
                        (s[i - 1] == '(' || s[i - 1] == '+' || s[i - 1] == '-'))
                        nums.push(0);
                    // 新操作入栈，先计算栈内
                    // 只有满足栈内运算符高于或等于当前运算符才计算
                    while (!ops.empty() && ops.top() != '(') {
                        if (opMap[ops.top()] < opMap[c]) break;
                        calc();
                    }
                    ops.push(c);
                }
            }
        }
        while (!ops.empty()) calc();
        return nums.top();
    }
};
```

## 大数运算

### 大数相加

```c++
string sum(string a,string b) {
	if (a.size()<b.size()) 
		swap(a,b);
	int i,j;
	for (i=a.size()-1,j=b.size()-1; i>=0; i--,j--) {	
		a[i]=(char)(a[i]+(j>=0?b[j]-'0':0));
		if (a[i]-'0'>=10) {
			a[i]=(char)((a[i]-'0')%10+'0');
			if(i) a[i-1]++;
			else a='1'+a;	
		}
	}
	return a;
}
```

### 大数相减

```c++
string differ(string a,string b) {
	int flag=0; //标记符号
	if (a.size()<b.size()) {
		swap(a,b);
		flag=1;
	} else if (a.size()==b.size()) {
		if (a<b) {
			swap(a,b);
			flag=1;
		}
	}
	for (int i=a.size()-1,j=b.size()-1; i>=0; i--,j--) {
		if (j>=0) b[i]=b[j];
		else b[i]='0'; //给减数不足的位数补0
	}
	for (int i=a.size()-1; i>=0; i--) {	//模拟减法操作
		if (a[i]>=b[i])
			a[i]=a[i]-b[i]+'0';
		else {
			a[i]=a[i]-b[i]+10+'0';
			a[i-1]--;
		}
	}
	while (a[0]=='0' && a.size()!=1) {	//去掉开头的0，如果全为0留一个
		a.erase(0,1);
	}
	if (flag) a='-'+a;
	return a;
}
```

### 大数相乘

```c++
string product(string a, string b) {
	string ans="0";
	if (a.size()<b.size())
		swap(a,b);
	int cnt=-1;
	for (int i=b.size()-1,j=a.size()-1; i>=0; i--){
		int t=j;
		string tmp;
		int next=0;	//表示进位
		while (t>=0){	//循环得出中间的数
			int mid=(b[i]-'0')*(a[t]-'0')+next;	//存储中间的乘积
			tmp=(char)(mid%10+'0')+tmp;
			if (mid>=10)		//乘积大于10则进位
				next=mid/10;
			else next=0;
			t--;
		}
		if (next!=0) tmp=(char)(next+'0')+tmp;		//如果循环结束进位不为0，则在前面补
		cnt++;
		for (int k=0; k<cnt; k++)	//错位相加时补0
			tmp=tmp+'0';
		ans=sum(ans,tmp);
		tmp.clear();	//将存储清空
	}
	while (ans[0]=='0' && ans.size()!=1)	//去掉前面多余的0
		ans.erase(0,1);	
	return ans;
}
```
