# 泛算法：关于动态规划的一些文字

# Some Words About Dynamic Programming

## 2021 年 4月 24 日

## Apr 24 2021

---

### 引言

在数学上，当一个函数可以接受函数为变量或产生函数时，我们称这个函数为泛函。动态规划就像这样一种算法，用动态规划的思想可以产生一系列算法，由此，我们可以称动态规划为「泛算法」；[《挑战程序设计竞赛》](https://book.douban.com/subject/24749842/)总结动态规划的思想为“记录结果的再利用”，这个说法具有普遍意义，但是没有提供足够具体的信息。动态规划算法一定要结合具体的题型来分析；动态规划看多了，容易造成“看什么问题都像动态规划”的错觉，在学习时一定要避免这种<u>魔怔</u>思想。

### 经典问题集合：背包问题（对限制条件规划）

力扣选手[润山](https://leetcode-cn.com/problems/combination-sum-iv/solution/xi-wang-yong-yi-chong-gui-lu-gao-ding-bei-bao-wen-/)曾总结过背包问题的三个判定法则：

1. 如果是<u>0-1背包</u>，那么nums放在外循环，target在内循环，且内循环<u>倒序</u>；

   ```javascript
   for (const num of nums) {
     for (let i=target ; i>=0; --i) {
       ...
     }
   }
   ```

   倒叙的原因正是「元素不可重复选取」。

2. 如果是<u>完全背包</u>，同样nums放在外循环，target在内循环，但内循环<u>正序</u>；

   ```javascript
   for (const num of nums) {
     for (let i=num; i<target+1; ++i) {
       ...
     }
   }
   ```

3. 如果是<u>顺序敏感</u>问题（元素可以按不同顺序排列），需将target放在外循环，将nums放在内循环；

   ```javascript
   for (let i=1; i<target+1; ++i) {
     for (const num of nums) {
       ...
     }
   }
   ```

至于省略号里应该填什么，就涉及背包问题的问题类型，详情见下文。

背包问题用动态规划方法来求解，一维数组初始化为`target`的长度加一，

```javascript
const dp = new Array(target+1).fill(0);
dp[0] = 1;
```

1. 组合问题

   ```javascript
   if (i >= num) {
     dp[i] += dp[i-num]
   }
   ```

2. True、False问题

   ```
   dp[i] = dp[i] || dp[i-num]
   ```

3. 最大最小问题

   ```javascript
   if (i >= num) {
     dp[i] = min(dp[i], dp[i-num]+1)
   	dp[i] = max(dp[i], dp[i-num]+1)
   }
   ```

背包问题是动态规划极经典的问题之一，其问题类型数量之庞大，覆盖面之广。

这里推荐阅读[崔添翼](https://www.linkedin.com/in/tianyicui/)同学的《背包九讲》，是一篇深入分析背包问题的长文。

### 经典单串：LIS

这个问题的思想可以延伸到许多问题，务必牢记。

问题在于：什么时候可以用二分搜索优化？笔者还不知道答案，所以这里留白，以后若想明白了在这里补充上。

### 经典区间规划：回文子串（单串、倒序、区间、二维dp）

如何快速判断连续一段` [i, j]` 是否为回文串？

一个直观的做法是，我们先预处理除所有的 `f[i][j]`，`f[i][j]` 代表 `[i, j]` 这一段是否为回文串。

要想` f[i][j] == true` ，必须满足以下两个条件：

1. `f[i + 1][j - 1] == true`
2. `s[i] == s[j]`

<u>重点：</u>

- 状态` f[i][j]` 依赖于状态` f[i + 1][j - 1]`；
- 左端点` i `是<u>从大到小</u>进行遍历；而右端点` j `是<u>从小到大</u>进行遍历；

因此，我们的遍历过程可以整理为：左端点` i `从大到小移动，在` i `固定情况下，右端点` j `从` i `的左边开始，一直往右移动（从小到大）。

```JavaScript
const f = new Array(n).fill(0).map(() => new Array(n).fill(true));
for (let i=n-2; i>=0; --i) {
  for (let j=i+1; j<n; ++j) {
    f[i][j] = (s[i] === s[j]) && (f[i+1][j-1])
  }
}
```

### 经典双串：LCS



练习题：

- [编辑距离](https://leetcode-cn.com/problems/edit-distance/)
- [Distinct Subsequences](https://leetcode-cn.com/problems/distinct-subsequences/)
- [交错字符串](https://leetcode-cn.com/problems/interleaving-string/)

