---
title: 回溯算法
tags:
  - Backtracking
categories: [Computer Basics,Algorithm]
toc: true
date: 2021-12-26 13:15
updated: 2021-12-26 13:15
references:
  - title: 代码随想录
    url: https://programmercarl.com/%E5%9B%9E%E6%BA%AF%E7%AE%97%E6%B3%95%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80.html
---

**回溯的本质是穷举，穷举所有可能，然后选出想要的答案**，如果想让回溯法高效一些，需要进行**剪枝**操作。

回溯法，一般可以解决如下几种问题：

- 组合问题：N 个数里面按一定规则找出 k 个数的集合
- 切割问题：一个字符串按一定规则有几种切割方式
- 子集问题：一个 N 个数的集合里有多少符合条件的子集
- 排列问题：N 个数按一定规则全排列，有几种排列方式
- 棋盘问题：N 皇后，解数独等等

> **组合不强调元素顺序，排列强调元素顺序**：
>
> 即 不同顺序的同样元素集合 算作排列，但不算组合

<!-- more -->

## 前言

看到回溯，感觉和 DFS（深度优先搜索）区别不太大，两者其实是包含关系。

回溯搜索是 DFS 的一种，对于某一个搜索树来说（搜索树记录路径和状态判断），其主要的区别是：

- 回溯法在求解过程中不保留完整的树结构，而深度优先搜索则记下完整的搜索树
- 为了减少存储空间，在深度优先搜索中，用标志的方法记录访问过的状态，这种方法使得深度优先搜索法与回溯法没什么区别

## 组合问题

> N 个数里面按一定规则找出 k 个数的集合。

直接 DFS 暴搜，如果满足条件返回，再加上适当剪枝即可。

> 如果是一个集合来求组合的话，就需要 startIndex，例如：[回溯算法：求组合问题！ (opens new window)](https://programmercarl.com/0077.%E7%BB%84%E5%90%88.html)，[回溯算法：求组合总和！ (opens new window)](https://programmercarl.com/0216.%E7%BB%84%E5%90%88%E6%80%BB%E5%92%8CIII.html)。
> 如果是多个集合取组合，各个集合之间相互不影响，那么就不用 startIndex，例如：[回溯算法：电话号码的字母组合](https://programmercarl.com/0017.%E7%94%B5%E8%AF%9D%E5%8F%B7%E7%A0%81%E7%9A%84%E5%AD%97%E6%AF%8D%E7%BB%84%E5%90%88.html)

例题：[77. 组合 - 力扣（LeetCode）](https://leetcode-cn.com/problems/combinations/)

```c++
class Solution {
   public:
    vector<vector<int>> ans;
    vector<int> tmp;

    void dfs(int n, int k, int idx) {
        if (tmp.size() == k) {
            ans.push_back(tmp);
            return;
        }
        for (int i = idx; i <= n - (k - tmp.size()) + 1; i++) {
            tmp.push_back(i);
            dfs(n, k, i + 1);
            tmp.pop_back();
        }
    }

    vector<vector<int>> combine(int n, int k) {
        dfs(n, k, 1);
        return ans;
    }
};
```

## 切割问题

> 一个字符串按一定规则有几种切割方式

具体算法类似组合问题，判断函数比较复杂。

例题：[131. 分割回文串 - 力扣（LeetCode）](https://leetcode-cn.com/problems/palindrome-partitioning/)

```c++
class Solution {
   public:
    vector<vector<string>> ans;
    vector<string> tmp;

    bool check(string s, int start, int end) {
        for (int i = start, j = end; i < j; i++, j--) {
            if (s[i] != s[j]) return false;
        }
        return true;
    }

    void dfs(string s, int idx) {
        if (idx >= s.size()) {
            ans.push_back(tmp);
            return;
        }
        for (int i = idx; i < s.size(); i++) {
            if (!check(s, idx, i)) continue;
            tmp.push_back(s.substr(idx, i - idx + 1));
            dfs(s, i + 1);
            tmp.pop_back();
        }
    }

    vector<vector<string>> partition(string s) {
        dfs(s, 0);
        return ans;
    }
};
```

## 子集问题

> 一个 N 个数的集合里有多少符合条件的子集

如果把 子集问题、组合问题、分割问题都抽象为一棵树的话，**那么组合问题和分割问题都是收集树的叶子节点，而子集问题是找树的所有节点。**

例题：[78. 子集 - 力扣（LeetCode）](https://leetcode-cn.com/problems/subsets/)

```c++
class Solution {
   public:
    vector<vector<int>> ans;
    vector<int> tmp;

    void dfs(vector<int>& nums, int idx) {
        ans.push_back(tmp);
        if (idx >= nums.size()) return;
        for (int i = idx; i < nums.size(); i++) {
            tmp.push_back(nums[i]);
            dfs(nums, i + 1);
            tmp.pop_back();
        }
    }

    vector<vector<int>> subsets(vector<int>& nums) {
        dfs(nums, 0);
        return ans;
    }
};
```

## 排列问题

> N 个数按一定规则全排列，有几种排列方式
>
> 排列是区分顺序的，不同顺序的集合算不同排列

和组合问题区别在每次循环的起始位置都是 0，同时用 vis 数组来记录状态。

```c++
class Solution {
   public:
    vector<vector<int>> ans;
    vector<int> tmp;

    void dfs(vector<int>& nums, vector<bool>& vis) {
        if (tmp.size() == nums.size()) {
            ans.push_back(tmp);
            return;
        }
        for (int i = 0; i < nums.size(); i++) {
            if (vis[i]) continue;
            tmp.push_back(nums[i]);
            vis[i] = true;
            dfs(nums, vis);
            vis[i] = false;
            tmp.pop_back();
        }
    }

    vector<vector<int>> permute(vector<int>& nums) {
        vector<bool> vis(nums.size(), false);
        dfs(nums, vis);
        return ans;
    }
};
```

## 棋盘问题

> N 皇后，解数独等等

给回溯函数加上 bool 返回值，找到一组成功解则返回。

例题：[51. N 皇后 - 力扣（LeetCode）](https://leetcode-cn.com/problems/n-queens/)

```c++
class Solution {
   public:
    vector<vector<string>> ans;

    /*
     * 递归遍历行
     * 循环遍历列
     */
    void dfs(int row, int n, vector<string>& chess) {
        if (row == n) {
            ans.push_back(chess);
            return;
        }
        for (int col = 0; col < n; col++) {
            if (check(row, col, n, chess)) {
                chess[row][col] = 'Q';
                dfs(row + 1, n, chess);
                chess[row][col] = '.';
            }
        }
    }

    /*
     *不能同行（递归过程中进行了同行检查）
     *不能同列
     *不能同斜线 （45度和135度角）
     */
    bool check(int row, int col, int n, vector<string>& chess) {
        // 检查列，剪枝
        for (int i = 0; i < row; i++) {
            if (chess[i][col] == 'Q') return false;
        }
        // ∠45°
        for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
            if (chess[i][j] == 'Q') return false;
        }
        // ∠135°
        for (int i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++) {
            if (chess[i][j] == 'Q') return false;
        }
        return true;
    }

    vector<vector<string>> solveNQueens(int n) {
        vector<string> chess(n, string(n, '.'));
        dfs(0, n, chess);
        return ans;
    }
};
```

例题：[37. 解数独 - 力扣（LeetCode）](https://leetcode-cn.com/problems/sudoku-solver/)

```c++
class Solution {
   public:
    bool check(int row, int col, int val, vector<vector<char>>& board) {
        for (int j = 0; j < 9; j++) {  // 检查列
            if (board[row][j] == val) return false;
        }
        for (int i = 0; i < 9; i++) {  // 检查行
            if (board[i][col] == val) return false;
        }
        // 检查 3x3
        int nRow = (row / 3) * 3;
        int nCol = (col / 3) * 3;
        for (int i = nRow; i < nRow + 3; i++) {
            for (int j = nCol; j < nCol + 3; j++)
                if (board[i][j] == val) return false;
        }
        return true;
    }

    bool dfs(vector<vector<char>>& board) {
        for (int i = 0; i < board.size(); i++) {
            for (int j = 0; j < board[0].size(); j++) {
                if (board[i][j] != '.') continue;
                for (char k = '1'; k <= '9'; k++) {
                    if (check(i, j, k, board)) {
                        board[i][j] = k;
                        if (dfs(board)) return true;
                        board[i][j] = '.';
                    }
                }
                return false;  // 9 个数都不行
            }
        }
        return true;
    }

    void solveSudoku(vector<vector<char>>& board) { dfs(board); }
};
```
