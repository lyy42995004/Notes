## 中等

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

## [64. 最小路径和](https://leetcode.cn/problems/minimum-path-sum/)

> 给定一个包含非负整数的 `*m* x *n*` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。
>
> **说明：**每次只能向下或者向右移动一步。
>
>  
>
> **示例 1：**
>
> <img src="https://assets.leetcode.com/uploads/2020/11/05/minpath.jpg" alt="img" style="zoom:50%;" />
>
> ```
> 输入：grid = [[1,3,1],[1,5,1],[4,2,1]]
> 输出：7
> 解释：因为路径 1→3→1→1→1 的总和最小。
> ```
>
> **示例 2：**
>
> ```
> 输入：grid = [[1,2,3],[4,5,6]]
> 输出：12
> ```
>
>  
>
> **提示：**
>
> - `m == grid.length`
> - `n == grid[i].length`
> - `1 <= m, n <= 200`
> - `0 <= grid[i][j] <= 200`

```cpp
int minPathSum(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    // 到grid[i-1][j-1]位置的最小路径和
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, INT_MAX));
    dp[1][0] = 0; // 保证dp[1][1]为grid[0][0]

    for (int i = 1; i <= m; ++i) {
        for (int j = 1; j <= n; ++j) {
            dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i - 1][j - 1];
        }
    }

    return dp[m][n];
}
```

## [931. 下降路径最小和](https://leetcode.cn/problems/minimum-falling-path-sum/)

> 给你一个 `n x n` 的 **方形** 整数数组 `matrix` ，请你找出并返回通过 `matrix` 的**下降路径** 的 **最小和** 。
>
> **下降路径** 可以从第一行中的任何元素开始，并从每一行中选择一个元素。在下一行选择的元素和当前行所选元素最多相隔一列（即位于正下方或者沿对角线向左或者向右的第一个元素）。具体来说，位置 `(row, col)` 的下一个元素应当是 `(row + 1, col - 1)`、`(row + 1, col)` 或者 `(row + 1, col + 1)` 。
>
>  
>
> **示例 1：**
>
> <img src="https://pic.leetcode.cn/1729566253-aneDag-image.png" alt="img" style="zoom:50%;" />
>
> ```
> 输入：matrix = [[2,1,3],[6,5,4],[7,8,9]]
> 输出：13
> 解释：如图所示，为和最小的两条下降路径
> ```
>
> **示例 2：**
>
> <img src="https://pic.leetcode.cn/1729566282-dtXwRd-image.png" alt="img" style="zoom:50%;" />
>
> ```
> 输入：matrix = [[-19,57],[-40,-5]]
> 输出：-59
> 解释：如图所示，为和最小的下降路径
> ```
>
>  
>
> **提示：**
>
> - `n == matrix.length == matrix[i].length`
> - `1 <= n <= 100`
> - `-100 <= matrix[i][j] <= 100`

```cpp
int minFallingPathSum(vector<vector<int>>& matrix) {
    int n = matrix.size();
    // 到matrix[i-1][j-1]的最小路径和
    vector<vector<int>> dp(n + 1, vector<int>(n + 2));
    for (int i = 0; i < n; ++i) {
        dp[i][0] = dp[i][n+1] = INT_MAX;
    }

    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= n; ++j) {
            dp[i][j] = min(dp[i-1][j], min(dp[i-1][j-1], dp[i-1][j+1]));
            dp[i][j] += matrix[i-1][j-1];
        }
    }

    int ans = INT_MAX;
    for (int j = 1; j <= n; ++j) {
        ans = min(ans, dp[n][j]);
    }
    return ans;
}
```

## [LCR 166. 珠宝的最高价值](https://leetcode.cn/problems/li-wu-de-zui-da-jie-zhi-lcof/)

> 现有一个记作二维矩阵 `frame` 的珠宝架，其中 `frame[i][j]` 为该位置珠宝的价值。拿取珠宝的规则为：
>
> - 只能从架子的左上角开始拿珠宝
> - 每次可以移动到右侧或下侧的相邻位置
> - 到达珠宝架子的右下角时，停止拿取
>
> 注意：珠宝的价值都是大于 0 的。除非这个架子上没有任何珠宝，比如 `frame = [[0]]`。
>
>  
>
> **示例 1：**
>
> ```
> 输入：frame = [[1,3,1],[1,5,1],[4,2,1]]
> 输出：12
> 解释：路径 1→3→5→2→1 可以拿到最高价值的珠宝
> ```
>
>  
>
> **提示：**
>
> - `0 < frame.length <= 200`
> - `0 < frame[0].length <= 200`

```cpp
int jewelleryValue(vector<vector<int>>& frame) {
    int m = frame.size(), n = frame[0].size();
    // 到frame[i-1][j-1]位置的最大珠宝价值
    vector<vector<int>> dp(m + 1, vector<int>(n + 1));
    for (int i = 1; i <= m; ++i) {
        for (int j = 1; j <= n; ++j) {
            dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]) + frame[i - 1][j - 1];
        }
    }
    return dp[m][n];
}
```

# 困难

## [174. 地下城游戏](https://leetcode.cn/problems/dungeon-game/)

> 恶魔们抓住了公主并将她关在了地下城 `dungeon` 的 **右下角** 。地下城是由 `m x n` 个房间组成的二维网格。我们英勇的骑士最初被安置在 **左上角** 的房间里，他必须穿过地下城并通过对抗恶魔来拯救公主。
>
> 骑士的初始健康点数为一个正整数。如果他的健康点数在某一时刻降至 0 或以下，他会立即死亡。
>
> 有些房间由恶魔守卫，因此骑士在进入这些房间时会失去健康点数（若房间里的值为*负整数*，则表示骑士将损失健康点数）；其他房间要么是空的（房间里的值为 *0*），要么包含增加骑士健康点数的魔法球（若房间里的值为*正整数*，则表示骑士将增加健康点数）。
>
> 为了尽快解救公主，骑士决定每次只 **向右** 或 **向下** 移动一步。
>
> 返回确保骑士能够拯救到公主所需的最低初始健康点数。
>
> **注意：**任何房间都可能对骑士的健康点数造成威胁，也可能增加骑士的健康点数，包括骑士进入的左上角房间以及公主被监禁的右下角房间。
>
>  
>
> **示例 1：**
>
> ![img](https://assets.leetcode.com/uploads/2021/03/13/dungeon-grid-1.jpg)
>
> ```
> 输入：dungeon = [[-2,-3,3],[-5,-10,1],[10,30,-5]]
> 输出：7
> 解释：如果骑士遵循最佳路径：右 -> 右 -> 下 -> 下 ，则骑士的初始健康点数至少为 7 。
> ```
>
> **示例 2：**
>
> ```
> 输入：dungeon = [[0]]
> 输出：1
> ```
>
>  
>
> **提示：**
>
> - `m == dungeon.length`
> - `n == dungeon[i].length`
> - `1 <= m, n <= 200`
> - `-1000 <= dungeon[i][j] <= 1000`

```cpp
int calculateMinimumHP(vector<vector<int>>& dungeon) {
    int m = dungeon.size(), n = dungeon[0].size();
    // 从dungeon[m-1][n-1]到[i][j]所需的最小初始生命值
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, INT_MAX));
    dp[m - 1][n] = 1;

    for (int i = m - 1; i >= 0; --i) {
        for (int j = n - 1; j >= 0; --j) {
            dp[i][j] = max(1, min(dp[i + 1][j], dp[i][j + 1]) - dungeon[i][j]);
        }
    }

    return dp[0][0];
}
```

