# 简单

## [70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/)

> 假设你正在爬楼梯。需要 `n` 阶你才能到达楼顶。
>
> 每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶呢？
>
>  
>
> **示例 1：**
>
> ```
> 输入：n = 2
> 输出：2
> 解释：有两种方法可以爬到楼顶。
> 1. 1 阶 + 1 阶
> 2. 2 阶
> ```
>
> **示例 2：**
>
> ```
> 输入：n = 3
> 输出：3
> 解释：有三种方法可以爬到楼顶。
> 1. 1 阶 + 1 阶 + 1 阶
> 2. 1 阶 + 2 阶
> 3. 2 阶 + 1 阶
> ```
>
>  
>
> **提示：**
>
> - `1 <= n <= 45`

```cpp
int climbStairs(int n) {
    if (n == 1 || n == 2) {
        return n;
    }
    int a = 1, b = 2, c;
    for (int i = 2; i < n; ++i) {
        c = a + b;
        a = b;
        b = c;
    }
    return c;
}
```

## [746. 使用最小花费爬楼梯](https://leetcode.cn/problems/min-cost-climbing-stairs/)

> 给你一个整数数组 `cost` ，其中 `cost[i]` 是从楼梯第 `i` 个台阶向上爬需要支付的费用。一旦你支付此费用，即可选择向上爬一个或者两个台阶。
>
> 你可以选择从下标为 `0` 或下标为 `1` 的台阶开始爬楼梯。
>
> 请你计算并返回达到楼梯顶部的最低花费。
>
>  
>
> **示例 1：**
>
> ```
> 输入：cost = [10,15,20]
> 输出：15
> 解释：你将从下标为 1 的台阶开始。
> - 支付 15 ，向上爬两个台阶，到达楼梯顶部。
> 总花费为 15 。
> ```
>
> **示例 2：**
>
> ```
> 输入：cost = [1,100,1,1,1,100,1,1,100,1]
> 输出：6
> 解释：你将从下标为 0 的台阶开始。
> - 支付 1 ，向上爬两个台阶，到达下标为 2 的台阶。
> - 支付 1 ，向上爬两个台阶，到达下标为 4 的台阶。
> - 支付 1 ，向上爬两个台阶，到达下标为 6 的台阶。
> - 支付 1 ，向上爬一个台阶，到达下标为 7 的台阶。
> - 支付 1 ，向上爬两个台阶，到达下标为 9 的台阶。
> - 支付 1 ，向上爬一个台阶，到达楼梯顶部。
> 总花费为 6 。
> ```
>
>  
>
> **提示：**
>
> - `2 <= cost.length <= 1000`
> - `0 <= cost[i] <= 999`

```cpp
int minCostClimbingStairs(vector<int>& cost) {
    int n = cost.size();
    // 爬到i位置的最小花费
    vector<int> dp(n + 1);
    dp[0] = 0, dp[1] = 0;
    for (int i = 2; i <= n; ++i) {
        dp[i] = min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2]);
    }
    return dp[n];
}
```

## [1137. 第 N 个泰波那契数](https://leetcode.cn/problems/n-th-tribonacci-number/)

> 泰波那契序列 Tn 定义如下： 
>
> T0 = 0, T1 = 1, T2 = 1, 且在 n >= 0 的条件下 Tn+3 = Tn + Tn+1 + Tn+2
>
> 给你整数 `n`，请返回第 n 个泰波那契数 Tn 的值。
>
>  
>
> **示例 1：**
>
> ```
> 输入：n = 4
> 输出：4
> 解释：
> T_3 = 0 + 1 + 1 = 2
> T_4 = 1 + 1 + 2 = 4
> ```
>
> **示例 2：**
>
> ```
> 输入：n = 25
> 输出：1389537
> ```
>
>  
>
> **提示：**
>
> - `0 <= n <= 37`
> - 答案保证是一个 32 位整数，即 `answer <= 2^31 - 1`。

```cpp
int tribonacci(int n) 
{
    if (n == 0) 
        return 0;
    if (n == 1 || n == 2)
        return 1;
    vector<int> dp(n + 1);
    dp[0] = 0, dp[1] = dp[2] = 1;
    for (int i = 3; i <= n; ++i)
        dp[i] = dp[i - 1] + dp[i - 2] + dp[i - 3];
    return dp[n];
}
```

**优化**

```cpp
int tribonacci(int n) {
    if (n == 0) {
        return 0;
    }
    if (n == 1 || n == 2) {
        return 1;
    }
    int a = 0, b = 1, c = 1, d;
    for (int i = 3; i <= n; ++i) {
        d = a + b + c;
        a = b, b = c, c = d;
    }
    return d;
}
```

## [面试题 08.01. 三步问题](https://leetcode.cn/problems/three-steps-problem-lcci/)

> 三步问题。有个小孩正在上楼梯，楼梯有 n 阶台阶，小孩一次可以上 1 阶、2 阶或 3 阶。实现一种方法，计算小孩有多少种上楼梯的方式。结果可能很大，你需要对结果模 1000000007。
>
> **示例 1：**
>
> ```
>  输入：n = 3 
>  输出：4
>  说明：有四种走法
> ```
>
> **示例 2：**
>
> ```
>  输入：n = 5
>  输出：13
> ```
>
> **提示:**
>
> n 范围在[1, 1000000]之间

```cpp
    const int MOD = 1000000007;

    int waysToStep(int n) {
        int sz = max(3, n);
        vector<int> dp(sz + 1);
        dp[1] = 1, dp[2] = 2, dp[3] = 4;
        for (int i = 4; i <= n; ++i) {
            dp[i] = ((dp[i - 1] + dp[i - 2]) % MOD + dp[i - 3]) % MOD;
        }
        return dp[n];
    }
```

**优化**

```cpp
const int MOD = 1000000007;

int waysToStep(int n) {
    if (n == 1) {
        return 1;
    } else if (n == 2) {
        return 2;
    } if (n == 3) {
        return 4;
    }
    int a = 1, b = 2, c = 4, d;
    for (int i = 4; i <= n; ++i) {
        d = ((a + b) % MOD + c) % MOD;
        a = b, b = c, c = d;
    }
    return d;
}
```

# 中等


## [91. 解码方法](https://leetcode.cn/problems/decode-ways/)

> 一条包含字母 `A-Z` 的消息通过以下映射进行了 **编码** ：
>
> ```
> "1" -> 'A' "2" -> 'B' ... "25" -> 'Y' "26" -> 'Z'
> ```
>
> 然而，在 **解码** 已编码的消息时，你意识到有许多不同的方式来解码，因为有些编码被包含在其它编码当中（`"2"` 和 `"5"` 与 `"25"`）。
>
> 例如，`"11106"` 可以映射为：
>
> - `"AAJF"` ，将消息分组为 `(1, 1, 10, 6)`
> - `"KJF"` ，将消息分组为 `(11, 10, 6)`
> - 消息不能分组为 `(1, 11, 06)` ，因为 `"06"` 不是一个合法编码（只有 "6" 是合法的）。
>
> 注意，可能存在无法解码的字符串。
>
> 给你一个只含数字的 **非空** 字符串 `s` ，请计算并返回 **解码** 方法的 **总数** 。如果没有合法的方式解码整个字符串，返回 `0`。
>
> 题目数据保证答案肯定是一个 **32 位** 的整数。
>
>  
>
> **示例 1：**
>
> ```
> 输入：s = "12"
> 输出：2
> 解释：它可以解码为 "AB"（1 2）或者 "L"（12）。
> ```
>
> **示例 2：**
>
> ```
> 输入：s = "226"
> 输出：3
> 解释：它可以解码为 "BZ" (2 26), "VF" (22 6), 或者 "BBF" (2 2 6) 。
> ```
>
> **示例 3：**
>
> ```
> 输入：s = "06"
> 输出：0
> 解释："06" 无法映射到 "F" ，因为存在前导零（"6" 和 "06" 并不等价）。
> ```
>
>  
>
> **提示：**
>
> - `1 <= s.length <= 100`
> - `s` 只包含数字，并且可能包含前导零。

```cpp
int numDecodings(string s) {
    int n = s.size();
    // i结尾，字符串的解码方法总数
    vector<int> dp(n);
    
    int a = s[0] - '0', b = s[1] - '0';
    if (a < 10 && a > 0) {
        dp[0] = 1;
    }
    if (n == 1) {
        return dp[0];
    }
    
    if (10 * a + b >= 10 && 10 * a + b <= 26) {
        dp[1] = 1;
    }
    if (dp[0] != 0 && b < 10 && b > 0) {
        dp[1]++;
    }
    if (n == 2) {
        return dp[1];
    }
    
    for (int i = 2; i < n; ++i) {
        a = s[i - 1] - '0', b = s[i] - '0';
        if (b < 10 && b > 0) {
            dp[i] += dp[i - 1];
        }
        if (10 * a + b >= 10 && 10 * a + b <= 26)
            dp[i] += dp[i - 2];
    }
    return dp[n - 1];
}
```
