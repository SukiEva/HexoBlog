---
title: 动态规划
tags:
  - DP
categories: [Computer Basics,Algorithm]
toc: true
date: 2022-01-03 13:47
updated: 2022-01-03 13:47
references:
  - title: 代码随想录
    url: https://programmercarl.com/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80.html#%E4%BB%80%E4%B9%88%E6%98%AF%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92
---

动态规划（Dynamic Programming，简称 DP），如果某一问题有很多重叠子问题，使用动态规划是最有效的。

和贪心的区别在于：

- 动态规划中每一个状态一定是由上一个状态推导出来的
- 贪心没有状态推导，而是从局部直接选最优的，

<!-- more -->

卡子哥的**DP 五部曲**：

1. 确定 dp 数组（dp table）以及下标的含义
2. 确定递推公式
3. dp 数组如何初始化
4. 确定遍历顺序
5. 举例推导 dp 数组

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202201031418164.png" style="zoom: 50%;" />

## 背包问题

> 对于面试，掌握 01 背包和完全背包就够用了，最多可以再来一个多重背包。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202201061504469.png" style="zoom:80%;" />

### 01 背包

有 N 件物品和一个最多能背重量为 `W` 的背包。第 i 件物品的重量是 `weight[i]`，得到的价值是 `value[i]` 。

**每件物品只能用一次**，求解将哪些物品装入背包里物品价值总和最大。

#### 二维

1. `dp[i][j]`：从下标为 `[0-i]` 的物品里任意取，放进容量为 `j` 的背包，价值总和最大是多少
2. 有两个方向推出来 `dp[i][j]`：

- **不放物品 i**：由 `dp[i - 1][j]` 推出，即背包容量为 j，里面不放物品 i 的最大价值
- **放物品 i**：由 `dp[i - 1][j - weight[i]]` 推出，` dp[i - 1][j - weight[i]]  ` 为背包容量为 `j - weight[i]` 的时候不放物品 i 的最大价值，那么 `dp[i - 1][j - weight[i]] + value[i]` ，就是背包放物品 i 得到的最大价值
- 动态转移公式：$dp[i][j] = max(dp[i-1][j], dp[i-1][j-weight[i]]+value[i])$

3. 初始化：由递推公式看出，必须初始化 `dp[0][j]`（存放编号 0 的物品的时候，各个容量的背包所能存放的最大价值）

```c++
vector<vector<int>> dp(weight.size(), vector<int>(bagWeight + 1, 0));
for (int j = weight[0]; j <= bagWeight; j++) {
    dp[0][j] = value[0];
}
```

4. 遍历顺序：先遍历物品和先遍历背包重量都可，**先遍历物品**更好理解

```go
/* 01背包
- 动态转移方程：dp[i][j] = max(dp[i-1][j], dp[i-1][j-weight[i]]+value[i])
*/
func bagProblem(weight, value []int, bagWeight int) int {
	dp := make([][]int, len(weight))
	for i := range dp {
		dp[i] = make([]int, bagWeight+1)
	}
	// 初始化, dp[0][j]
	for j := weight[0]; j <= bagWeight; j++ {
		dp[0][j] = value[0]
	}
	for i := 1; i < len(weight); i++ {
		for j := 0; j <= bagWeight; j++ {
			if j < weight[i] { // 放不下
				dp[i][j] = dp[i-1][j]
			} else {
				dp[i][j] = max(dp[i-1][j], dp[i-1][j-weight[i]]+value[i])
			}
		}
	}
	return dp[len(weight)-1][bagWeight]
}
```

#### 一维

滚动数组：把二维 dp 降为一维 dp

> 对于背包问题其实状态都是可以压缩的。
>
> 在使用二维数组的时候，递推公式：`dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i])`
>
> **其实可以发现如果把 dp[i - 1] 那一层拷贝到 dp[i] 上，表达式完全可以是：`dp[i][j] = max(dp[i][j], dp[i][j - weight[i]] + value[i])`**
>
> **与其把 dp[i - 1] 这一层拷贝到 dp[i] 上，不如只用一个一维数组了**，只用 dp[j]（一维数组，也可以理解是一个滚动数组）
>
> **倒叙遍历是为了保证物品 i 只被放入一次**

```go
// 滚动数组优化 一维
// - dp[j] = max(dp[j], dp[j - weight[i]] + value[i])
func bagProblemBetter(weight, value []int, bagWeight int) int {
	dp := make([]int, bagWeight+1)
	for i := range weight {
		for j := bagWeight; j >= weight[i]; j-- { // 倒序，正序会状态重复
			dp[j] = max(dp[j], dp[j-weight[i]]+value[i])
		}
	}
	return dp[bagWeight]
}
```

### 完全背包

有 `N` 件物品和一个最多能背重量为 `W` 的背包。第 i 件物品的重量是 `weight[i]`，得到的价值是 `value[i]` 。

**每件物品能用无数次**，求解将哪些物品装入背包里物品价值总和最大。

> **01 背包和完全背包唯一不同就是体现在遍历顺序上：**
>
> - 01 背包内嵌的循环是从大到小遍历，为了保证每个物品仅被添加一次。
>
> - 完全背包的物品是可以添加多次的，所以要从小到大去遍历
>
> **为什么遍历物品在外层循环，遍历背包容量在内层循环？**
>
> 在完全背包中，对于一维 dp 数组来说，其实两个 for 循环嵌套顺序同样无所谓

```go
// 先遍历物品, 再遍历背包
// ! 如果求组合数就是外层for循环遍历物品，内层for遍历背包。
func bagProblem(weight, value []int, bagWeight int) int {
	dp := make([]int, bagWeight+1)
	for i := 0; i < len(weight); i++ {
		for j := weight[i]; j <= bagWeight; j++ {
			dp[j] = max(dp[j], dp[j-weight[i]]+value[i])
		}
	}
	return dp[bagWeight]
}

// 先遍历背包, 再遍历物品
// ! 如果求排列数就是外层for遍历背包，内层for循环遍历物品
func bagProblem2(weight, value []int, bagWeight int) int {
	dp := make([]int, bagWeight+1)
	for j := 0; j <= bagWeight; j++ {
		for i := 0; i < len(weight); i++ {
			if j >= weight[i] {
				dp[j] = max(dp[j], dp[j-weight[i]]+value[i])
			}
		}
	}
	return dp[bagWeight]
}
```

### 多重背包

有 `N` 种物品和一个容量为 `V` 的背包。第 `i` 种物品最多有 $M_i$ 件可用，每件耗费的空间是 $C_i$ ，价值是 $W_i$ 。
求解将哪些物品装入背包可使这些物品的耗费的空间总和不超过背包容量，且价值总和最大。

> 每件物品最多有 $M_i$ 件可用，把 $M_i$ 件摊开，其实就是一个 01 背包问题了

```go
// ! 转换成 01 背包
// - O(m*n*k)
func bagProblem(weight, value, nums []int, bagweight int) int {
	dp := make([]int, bagweight+1)
	for i := range weight {
		for j := bagweight; j >= weight[i]; j-- {
			// 遍历背包个数
			for k := 1; k <= nums[i] && j-k*weight[i] >= 0; k++ {
				dp[j] = max(dp[j], dp[j-k*weight[i]]+k*value[i])
			}
		}
		//fmt.Println(dp)
	}
	return dp[bagweight]
}
```

## 打家劫舍

### 一条边

> 你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。
>
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 **不触动警报装置的情况下** ，一夜之内能够偷窃到的最高金额。

**dp[i]：考虑下标 i（包括 i）以内的房屋，最多可以偷窃的金额为 dp[i]**

那么需要考虑 2 种情况：

- i 偷：$dp[i]=dp[i-2]+nums[i]$
- i 不偷：$dp[i]=dp[i-1]$

所以动态转移公式为：$dp[i]=max(dp[i-2]+nums[i],dp[i-1])$

```c++
class Solution {
   public:
    int rob(vector<int>& nums) {
        if (nums.size() == 1) return nums[0];
        vector<int> dp(nums.size());
        dp[0] = nums[0];
        dp[1] = max(nums[0], nums[1]);
        for (int i = 2; i < nums.size(); i++) {
            dp[i] = max(dp[i - 2] + nums[i], dp[i - 1]);
        }
        return dp[nums.size() - 1];
    }
};
```

### 记录路径

类似股票问题，多开 2 个状态：

- `dp[i][0]`：不偷 i
- `dp[i][1]`：偷 i

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        if (nums.size()<2) return nums[0];
        int n = nums.size();
        vector<vector<int>> dp(n,vector<int>(2));
        dp[0][1]=nums[0];
        for (int i=1; i<n; i++){
            dp[i][0] = max(dp[i-1][1],dp[i-1][0]);
            dp[i][1] = dp[i-1][0]+nums[i];
        }
        int target = max(dp[n-1][0],dp[n-1][1]);
        vector<int> path;
        for (int i=dp.size()-1; i>=0; i--){
            if (dp[i][1]==target){
                path.push_back(nums[i]);
                target-=nums[i];
            }
        }
        for (int i=path.size()-1; i>=0; i--)
            cout<<path[i]<<" ";
        return max(dp[n-1][0],dp[n-1][1]);
    }
};
```

### 围成圈

> 在一条边的基础上，收尾相连形成圈

只要比较 首和尾存一个 的两种情况即可。

```c++
class Solution {
   public:
    int rob(vector<int>& nums) {
        if (nums.size() == 1) return nums[0];
        return max(robrob(nums, 0, nums.size() - 2),
                   robrob(nums, 1, nums.size() - 1));
    }

    int robrob(vector<int>& nums, int start, int end) {
        if (start == end) return nums[start];
        vector<int> dp(nums.size());
        dp[start] = nums[start];
        dp[start + 1] = max(nums[start], nums[start + 1]);
        for (int i = start + 2; i <= end; i++) {
            dp[i] = max(dp[i - 2] + nums[i], dp[i - 1]);
        }
        return dp[end];
    }
};
```

### 形成树

> 变成一棵二叉树，树形 dp
>
> ```
>
> 输入: [3,2,3,null,3,null,1]
>
>      3
>     / \
>    2   3
>     \   \ 
>      3   1
>
> 输出: 7 
> 解释: 小偷一晚能够盗取的最高金额 = 3 + 3 + 1 = 7.
> ```

dp 数组记录 2 个状态：

- dp[0]：记录不偷该节点所得到的的最大金钱
- dp[1]：记录偷该节点所得到的的最大金钱

> 长度为 2 的数组怎么标记树中每个节点的状态呢？
>
> 在递归的过程中，系统栈会保存每一层递归的参数

左右根顺序（后序）遍历，那么分为 2 种情况：

- 根偷：两个子结点不偷 $  = left[0] + right[0] + root->val $
- 根不偷：考虑两个子结点偷 $  = max(left[0], left[1]) + max(right[0], right[1]) $

```c++
class Solution {
   public:
    int rob(TreeNode* root) {
        vector<int> ans = robTree(root);
        return max(ans[0], ans[1]);
    }

    vector<int> robTree(TreeNode* root) {
        if (root == nullptr) return {0, 0};
        vector<int> left = robTree(root->left);
        vector<int> right = robTree(root->right);
        int v1 = max(left[0], left[1]) + max(right[0], right[1]);
        int v2 = left[0] + right[0] + root->val;
        return {v1, v2};
    }
};
```

## 股票问题

主要通过二维数组记录各个状态，分析好每个状态的转移公式即可。

也可以滚动数组优化到一维，相当于覆盖前面的状态，不过不容易理解。

### 买卖 1 次

给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。

你只能选择 **某一天** 买入这只股票，并选择在 **未来的某一个不同的日子** 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0`

> 虽然有动态规划的思想，但第一感觉还是这样写

```c++
class Solution {
   public:
    int maxProfit(vector<int>& prices) {
        int minn = prices[0];
        int ans = 0;
        for (int i = 1; i < prices.size(); i++) {
            if (prices[i] - minn > 0) ans = max(ans, prices[i] - minn);
            minn = min(minn, prices[i]);
        }
        return ans;
    }
};
```

### 买卖多次

> 你可以尽可能地完成更多的交易

二维数组记录 2 个状态：

- `dp[i][0]` ：表示第 `i` 天持有股票所得最多现金
- `dp[i][1]` ：表示第 `i` 天不持有股票所得最多现金

```c++
class Solution {
   public:
    // dp[i][0] 持有股票
    // dp[i][1] 不持有股票
    int maxProfit(vector<int>& prices) {
        vector<vector<int>> dp(prices.size(), vector<int>(2));

        dp[0][0] = -prices[0];
        dp[0][1] = 0;
        for (int i = 1; i < prices.size(); i++) {
            dp[i][0] = max(dp[i - 1][1] - prices[i], dp[i - 1][0]);
            dp[i][1] = max(dp[i - 1][0] + prices[i], dp[i - 1][1]);
        }

        return dp[prices.size() - 1][1];
    }
};
```

### 最多买卖 2 次

> 你最多可以完成 **两笔** 交易。

二维数组记录 5 个状态：

- `dp[i][0]`：无操作
- `dp[i][1]`：第一次买入状态
- `dp[i][2]`：第一次卖出状态
- `dp[i][3]`：第二次买入状态
- `dp[i][4]`：第二次卖出状态

> 上面的状态不一定是第 i 天买入卖出，只是维持这种状态

```c++
class Solution {
   public:
    int maxProfit(vector<int>& prices) {
        vector<vector<int>> dp(prices.size(), vector<int>(5));
        dp[0][1] = -prices[0];
        dp[0][3] = -prices[0];
        for (int i = 1; i < prices.size(); i++) {
            dp[i][0] = dp[i - 1][0];
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
            dp[i][2] = max(dp[i - 1][2], dp[i - 1][1] + prices[i]);
            dp[i][3] = max(dp[i - 1][3], dp[i - 1][2] - prices[i]);
            dp[i][4] = max(dp[i - 1][4], dp[i - 1][3] + prices[i]);
        }
        return dp[prices.size() - 1][4];
    }
};
```

### 最多买卖 k 次

> 你最多可以完成 **k** 笔交易。

照着上面那个维护 $2*k+1$ 个状态即可

```c++
class Solution {
   public:
    int maxProfit(int k, vector<int>& prices) {
        if (prices.size() == 0) return 0;
        vector<vector<int>> dp(prices.size(), vector<int>(2 * k + 1));
        for (int i = 1; i <= 2 * k; i += 2) {
            dp[0][i] = -prices[0];
        }
        for (int i = 1; i < prices.size(); i++) {
            dp[i][0] = dp[i - 1][0];
            for (int j = 1; j <= 2 * k; j++) {
                if (j & 1)
                    dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - 1] - prices[i]);
                else
                    dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - 1] + prices[i]);
            }
        }
        return dp[prices.size() - 1][2 * k];
    }
};
```

### 含冷却期

> - 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）
> - 卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)

二维数组记录 4 个状态：

- `dp[i][0]`：卖出状态（非冷却期）
- `dp[i][1]`：买入状态
- `dp[i][2]`：卖出状态（刚卖出）
- `dp[i][3]`：卖出状态（冷却期）

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        vector<vector<int>> dp(n,vector<int>(4));
        dp[0][1]=-prices[0];
        for (int i=1; i<n; i++){
            dp[i][0] = max(dp[i-1][0],dp[i-1][3]); // （卖出）非冷却期
            dp[i][1] = max(dp[i-1][1],max(dp[i-1][0],dp[i-1][3])-prices[i]); // 买入
            dp[i][2] = dp[i-1][1]+prices[i]; // 刚卖出
            dp[i][3] = dp[i-1][2]; // 卖出冷却期
        }
        return max({dp[n-1][0],dp[n-1][2],dp[n-1][3]});
    }
};
```

### 含手续费

> 每笔交易你只需要为支付一次手续费。

跟买卖多次差不多，添上手续费即可

```c++
class Solution {
   public:
    int maxProfit(vector<int>& prices, int fee) {
        vector<vector<int>> dp(prices.size(), vector<int>(2));

        dp[0][0] = -prices[0];
        dp[0][1] = 0;
        for (int i = 1; i < prices.size(); i++) {
            dp[i][0] = max(dp[i - 1][1] - prices[i], dp[i - 1][0]);
            dp[i][1] = max(dp[i - 1][0] + prices[i] - fee, dp[i - 1][1]); // 这里减去手续费
        }
        return dp[prices.size() - 1][1];
    }
};
```

## 子序列问题

### 最长递增子序列

#### 子序列不连续

```c++
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> dp(nums.size(),1);
        int ans = 1;
        for (int i=1; i<nums.size(); i++){
            for (int j=0; j<i; j++){
                if (nums[j]<nums[i]){
                    dp[i] = max(dp[i],dp[j]+1);
                }
            }
            ans = max(dp[i],ans);
        }
        return ans;
    }
};

// 贪心 + 二分 nlogn
// 贪心：每次在上升子序列最后加上的那个数尽可能的小
class Solution {
   public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> dp(nums.size(), 1);  // 严格单调增，类似单调栈，更新栈顶
        int ans = 0;
        for (int num : nums) {
            int l = 0, r = ans; // 右边界 = 现有dp数组的长度
            while (l < r) { // 在现有的dp数组中查找比num大的数，用num取代它；如果没有，就添加入dp
                int m = (l + r) / 2;
                if (dp[m] < num) 
                    l = m + 1;
                else 
                    r = m;
            }
            dp[l] = num; // 保证最右边的数尽可能小
            if (ans == r) ans++; // dp数组中的数都比num小，添加入dp，长度+1
        }
        return ans;
    }
};
```

#### 记录路径

使用 dp 的话，本身就能倒推出路径；使用贪心 + 二分，记录的不一定是路径，模拟一下会发现小的会替代 dp 数组中的大值，导致顺序不对。

```c++
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp(n,1);
        int ans = 1;
        for (int i=1; i<n; i++){
            for (int j=0; j<i; j++){
                if (nums[j]<nums[i]) dp[i] = max(dp[i],dp[j]+1);
            }
             ans = max(ans,dp[i]);
        }
        int target = ans;
        vector<int> path;
        for (int i = dp.size()-1; i>=0; i--){
            if (dp[i]==target){
                path.push_back(nums[i]);
                target--;
            }
        }
        for (int i = path.size()-1; i>=0; i--)
            cout<<path[i]<<" ";
        return ans;
    }
};
```

#### 子序列连续

```c++
class Solution {
   public:
    int findLengthOfLCIS(vector<int>& nums) {
        int len = 1;
        int ans = 1;
        for (int i = 1; i < nums.size(); i++) {
            if (nums[i] > nums[i - 1]) len++;
            else len = 1;
            ans = max(ans, len);
        }
        return ans;
    }
};
```

### 公共子序列

#### 子序列不连续

> 给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长 **公共子序列** 的长度。如果不存在 **公共子序列** ，返回 `0` 。
>
> 一个字符串的 **子序列** 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
>
> - 例如，`"ace"` 是 `"abcde"` 的子序列，但 `"aec"` 不是 `"abcde"` 的子序列。
>
> 两个字符串的 **公共子序列** 是这两个字符串所共同拥有的子序列。

`dp[i][j]`：长度为 `[0, i - 1]` 的字符串 text1 与长度为 `[0, j - 1]` 的字符串 text2 的最长公共子序列

那么分为两种情况：

- `text1[i - 1]` 与 `text2[j - 1]` 相同
- `text1[i - 1]` 与 `text2[j - 1]` 不相同

```c++
class Solution {
   public:
    int longestCommonSubsequence(string text1, string text2) {
        vector<vector<int>> dp(text1.size() + 1, vector<int>(text2.size() + 1));
        for (int i = 1; i <= text1.size(); i++) {
            for (int j = 1; j <= text2.size(); j++) {
                if (text1[i - 1] == text2[j - 1])
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                else 
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
        return dp[text1.size()][text2.size()];
    }
};
```

> 类似题目：不相交的线
>
> 只是换了种题目说法，解法一模一样，转换过来就是最长公共子序列

#### 子序列连续

> 给两个整数数组 `A` 和 `B` ，返回两个数组中公共的、长度最长的子数组的长度。

`dp[i][j]`：以下标 `i - 1`**结尾**的字符串 A 与以下标 `j - 1`**结尾**的字符串 B 的最长重复子数组

那么只要考虑 `nums1[i - 1]` 与 `nums2[j - 1]` 相同时，更新长度

```c++
class Solution {
   public:
    int findLength(vector<int>& nums1, vector<int>& nums2) {
        vector<vector<int>> dp(nums1.size() + 1, vector<int>(nums2.size() + 1));
        int ans = 0;
        for (int i = 1; i <= nums1.size(); i++) {
            for (int j = 1; j <= nums2.size(); j++) {
                if (nums1[i - 1] == nums2[j - 1])
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                // else 无事发生，即 dp[i][j] = 0
                ans = max(ans, dp[i][j]);
            }
        }
        return ans;
    }
};
```

### 判断子序列

#### 单个

> 给定字符串 **s** 和 **t** ，判断 **s** 是否为 **t** 的子序列，即 s 是否在 t 子序列中出现

双指针更佳，dp 判断公共子序列修改下即可

```c++
class Solution {
public:
    bool isSubsequence(string s, string t) {
        int m = s.size(), n = t.size();
        vector<vector<int>> dp(m+1, vector<int>(n+1));
        for (int i=1; i<=m ;i++){
            for (int j=1; j<=n; j++){
                if (s[i-1]==t[j-1]) dp[i][j] = dp[i-1][j-1]+1;
                else dp[i][j] = dp[i][j-1];
            }
        }
        return dp[m][n] == m;
    }
};

// 双指针就行了
class Solution {
public:
    bool isSubsequence(string s, string t) {
        int n = s.length(), m = t.length();
        int i = 0, j = 0;
        while (i < n && j < m) {
            if (s[i] == t[j]) {
                i++;
            }
            j++;
        }
        return i == n;
    }
};
```

#### 多个

> 给定一个字符串 `s` 和一个字符串 `t` ，计算在 `s` 的子序列中 `t` 出现的个数

`dp[i][j]`：以 `i-1` 为结尾的 s 子序列中出现以 `j-1` 为结尾的 t 的个数

```c++
typedef unsigned long long ull;

class Solution {
   public:
    int numDistinct(string s, string t) {
        int m = s.size(), n = t.size();
        vector<vector<ull>> dp(m + 1, vector<ull>(n + 1));
        for (int i = 0; i < m; i++) dp[i][0] = 1;
        // for (int j=1; j<n; j++) dp[0][j] = 0;
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s[i - 1] == t[j - 1])
                    dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
                else
                    dp[i][j] = dp[i - 1][j];
            }
        }
        return dp[m][n];
    }
};
```

### 编辑距离

> 给你两个单词 `word1` 和 `word2`，请你计算出将 `word1` 转换成 `word2` 所使用的最少操作数 。
>
> 你可以对一个单词进行如下三种操作：
>
> - 插入一个字符
> - 删除一个字符
> - 替换一个字符

`dp[i][j]` 表示以下标 `i-1` 为结尾的字符串 word1，和以下标 `j-1` 为结尾的字符串 word2 的最近编辑距离

那么需要维护以下 4 种状态：

```c++
if (word1[i - 1] == word2[j - 1])
    不操作
if (word1[i - 1] != word2[j - 1])
    增
    删
    换
```

- 操作一：word1 删除一个元素，那么就是以下标 `i - 2` 为结尾的 word1 与 `j-1` 为结尾的 word2 的最近编辑距离 再加上一个操作

即 `dp[i][j] = dp[i - 1][j] + 1;`

- 操作二：word2 删除一个元素，那么就是以下标 `i - 1` 为结尾的 word1 与 `j-2` 为结尾的 word2 的最近编辑距离 再加上一个操作

即 `dp[i][j] = dp[i][j - 1] + 1;`

> word2 添加一个元素，相当于 word1 删除一个元素，即**增删等同**

- 操作三：替换元素，`word1` 替换 `word1[i - 1]`，使其与 `word2[j - 1]` 相同，此时不用增加元素，那么以下标 `i-2` 为结尾的 `word1` 与 `j-2` 为结尾的 `word2` 的最近编辑距离 加上一个替换元素的操作。

	即 `dp[i][j] = dp[i - 1][j - 1] + 1;`

```c++
class Solution {
   public:
    int minDistance(string word1, string word2) {
        vector<vector<int>> dp(word1.size() + 1, vector<int>(word2.size() + 1));
        for (int i = 0; i <= word1.size(); i++) dp[i][0] = i; // 删除 i 个
        for (int j = 0; j <= word2.size(); j++) dp[0][j] = j; // 删除 j 个
        for (int i = 1; i <= word1.size(); i++) {
            for (int j = 1; j <= word2.size(); j++) {
                if (word1[i - 1] == word2[j - 1])
                    dp[i][j] = dp[i - 1][j - 1];
                else
                    dp[i][j] = min({dp[i - 1][j - 1] + 1, dp[i - 1][j] + 1,
                                    dp[i][j - 1] + 1});
            }
        }
        return dp[word1.size()][word2.size()];
    }
};
```

### 回文

#### 回文子串

> 给你一个字符串 `s` ，请你统计并返回这个字符串中 **回文子串** 的数目。
>
> **回文字符串** 是正着读和倒过来读一样的字符串。
>
> **子字符串** 是字符串中的由连续字符组成的一个序列。
>
> 具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被视作不同的子串。

`dp[i][j]`：表示区间范围 `[i,j]` （左闭右闭）的子串是否是回文子串，如果是 `dp[i][j]` 为 true，否则为 false

- 当 s[i] 与 s[j] 不相等，`dp[i][j]` 一定是 false。
- 当 s[i] 与 s[j] 相等时，
	- 情况一：下标 i 与 j 相同，同一个字符例如 a，当然是回文子串
	- 情况二：下标 i 与 j 相差为 1，例如 aa，也是文子串
	- 情况三：下标 i 与 j 相差大于 1 的时候，例如 cabac，此时 s[i] 与 s[j] 已经相同了，我们看 i 到 j 区间是不是回文子串就看 aba 是不是回文就可以了，那么 aba 的区间就是 i+1 与 j-1 区间，这个区间是不是回文就看 `dp[i + 1][j - 1]` 是否为 true

**一定要从下到上，从左到右遍历，这样保证 `dp[i + 1][j - 1]` 都是经过计算的**

```c++
class Solution {
   public:
    int countSubstrings(string s) {
        int n = s.size();
        int ans = 0;
        vector<vector<bool>> dp(n, vector<bool>(n, false));
        for (int i = n - 1; i >= 0; i--) {
            for (int j = i; j < n; j++) {
                if (s[i] != s[j]) continue;
                if (j - i <= 1 || dp[i + 1][j - 1]) {
                    dp[i][j] = true;
                    ans++;
                }
            }
        }
        return ans;
    }
};
```

#### 回文子序列

> 给你一个字符串 `s` ，找出其中最长的回文子序列，并返回该序列的长度。
>
> 子序列定义为：不改变剩余字符顺序的情况下，删除某些字符或者不删除任何字符形成的一个序列。

`dp[i][j]`：字符串 s 在 `[i, j]` 范围内最长的回文子序列的长度

- 如果 s[i] 与 s[j] 相同，那么 $dp[i][j] = dp[i + 1][j - 1] + 2$
- 如果 s[i] 与 s[j] 不相同，那么 $dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])$

```c++
class Solution {
   public:
    int longestPalindromeSubseq(string s) {
        vector<vector<int>> dp(s.size(), vector<int>(s.size()));
        for (int i = 0; i < s.size(); i++) dp[i][i] = 1;
        for (int i = s.size() - 1; i >= 0; i--) {
            for (int j = i + 1; j < s.size(); j++) {
                if (s[i] == s[j])
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                else
                    dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);
            }
        }
        return dp[0][s.size() - 1];
    }
};
```
