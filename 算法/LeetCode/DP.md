# 简单

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

# 中等

## [62. 不同路径](https://leetcode.cn/problems/unique-paths/)

> 一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。
>
> 机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。
>
> 问总共有多少条不同的路径？
>
>  
>
> **示例 1：**
>
> ![img](https://pic.leetcode.cn/1697422740-adxmsI-image.png)
>
> ```
> 输入：m = 3, n = 7
> 输出：28
> ```
>
> **示例 2：**
>
> ```
> 输入：m = 3, n = 2
> 输出：3
> 解释：
> 从左上角开始，总共有 3 条路径可以到达右下角。
> 1. 向右 -> 向下 -> 向下
> 2. 向下 -> 向下 -> 向右
> 3. 向下 -> 向右 -> 向下
> ```
>
> **示例 3：**
>
> ```
> 输入：m = 7, n = 3
> 输出：28
> ```
>
> **示例 4：**
>
> ```
> 输入：m = 3, n = 3
> 输出：6
> ```
>
>  
>
> **提示：**
>
> - `1 <= m, n <= 100`
> - 题目数据保证答案小于等于 `2 * 10^9`

```cpp
int uniquePaths(int m, int n) {
    // 到[i][j]位置的路径数
    vector<vector<int>> dp(m, vector<int>(n));
    for (int i = 0; i < m; ++i) {
        dp[i][0] = 1;
    }
    for (int j = 0; j < n; ++j) {
        dp[0][j] = 1;
    }
    for (int i = 1; i < m; ++i) {
        for (int j = 1; j < n; ++j) {
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
        }
    }
    return dp[m - 1][n - 1];
}
```

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

## [63. 不同路径 II](https://leetcode.cn/problems/unique-paths-ii/)

> 给定一个 `m x n` 的整数数组 `grid`。一个机器人初始位于 **左上角**（即 `grid[0][0]`）。机器人尝试移动到 **右下角**（即 `grid[m - 1][n - 1]`）。机器人每次只能向下或者向右移动一步。
>
> 网格中的障碍物和空位置分别用 `1` 和 `0` 来表示。机器人的移动路径中不能包含 **任何** 有障碍物的方格。
>
> 返回机器人能够到达右下角的不同路径数量。
>
> 测试用例保证答案小于等于 `2 * 109`。
>
>  
>
> **示例 1：**
>
> <img src="https://assets.leetcode.com/uploads/2020/11/04/robot1.jpg" alt="img" style="zoom:50%;" />
>
> ```
> 输入：obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]
> 输出：2
> 解释：3x3 网格的正中间有一个障碍物。
> 从左上角到右下角一共有 2 条不同的路径：
> 1. 向右 -> 向右 -> 向下 -> 向下
> 2. 向下 -> 向下 -> 向右 -> 向右
> ```
>
> **示例 2：**
>
> <img src="https://assets.leetcode.com/uploads/2020/11/04/robot2.jpg" alt="img" style="zoom:50%;" />
>
> ```
> 输入：obstacleGrid = [[0,1],[0,0]]
> 输出：1
> ```
>
>  
>
> **提示：**
>
> - `m == obstacleGrid.length`
> - `n == obstacleGrid[i].length`
> - `1 <= m, n <= 100`
> - `obstacleGrid[i][j]` 为 `0` 或 `1`

```cpp
int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
    int m = obstacleGrid.size(), n = obstacleGrid[0].size();
    // 到[i][j]位置的路径数
    vector<vector<int>> dp(m, vector<int>(n));
    for (int i = 0; i < m; ++i) {
        if (obstacleGrid[i][0] == 1) {
            break;
        }
        dp[i][0] = 1;
    }
    for (int j = 0; j < n; ++j) {
        if (obstacleGrid[0][j] == 1) {
            break;
        }
        dp[0][j] = 1;
    }

    for (int i = 1; i < m; ++i) {
        for (int j = 1; j < n; ++j) {
            if (obstacleGrid[i][j] == 0) {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
    }
    return dp[m - 1][n - 1];
}
```

