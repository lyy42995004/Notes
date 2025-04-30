# 	简单

## [面试题 17.16. 按摩师](https://leetcode.cn/problems/the-masseuse-lcci/)

> 一个有名的按摩师会收到源源不断的预约请求，每个预约都可以选择接或不接。在每次预约服务之间要有休息时间，因此她不能接受相邻的预约。给定一个预约请求序列，替按摩师找到最优的预约集合（总预约时间最长），返回总的分钟数。
>
> **注意：**本题相对原题稍作改动
>
>  
>
> **示例 1：**
>
> ```
> 输入： [1,2,3,1]
> 输出： 4
> 解释： 选择 1 号预约和 3 号预约，总时长 = 1 + 3 = 4。
> ```
>
> **示例 2：**
>
> ```
> 输入： [2,7,9,3,1]
> 输出： 12
> 解释： 选择 1 号预约、 3 号预约和 5 号预约，总时长 = 2 + 9 + 1 = 12。
> ```
>
> **示例 3：**
>
> ```
> 输入： [2,1,4,5,3,1,1,3]
> 输出： 12
> 解释： 选择 1 号预约、 3 号预约、 5 号预约和 8 号预约，总时长 = 2 + 4 + 3 + 3 = 12。
> ```

```cpp
int massage(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) {
        return 0;
    }
    // dp[i][0]选到nums[i]的最长预约事件
    // dp[i][1]未选nums[i]的最长预约事件
    vector<vector<int>> dp(n, vector<int>(2));
    dp[0][0] = nums[0], dp[0][1] = 0;

    for (int i = 1; i < n; ++i) {
        dp[i][0] = dp[i-1][1] + nums[i];
        dp[i][1] = max(dp[i-1][0], dp[i-1][1]);
    }

    return max(dp[n-1][0], dp[n-1][1]);
}
```

# 中等

## [122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

> 给你一个整数数组 `prices` ，其中 `prices[i]` 表示某支股票第 `i` 天的价格。
>
> 在每一天，你可以决定是否购买和/或出售股票。你在任何时候 **最多** 只能持有 **一股** 股票。你也可以先购买，然后在 **同一天** 出售。
>
> 返回 *你能获得的 **最大** 利润* 。
>
>  
>
> **示例 1：**
>
> ```
> 输入：prices = [7,1,5,3,6,4]
> 输出：7
> 解释：在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4。
> 随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6 - 3 = 3。
> 最大总利润为 4 + 3 = 7 。
> ```
>
> **示例 2：**
>
> ```
> 输入：prices = [1,2,3,4,5]
> 输出：4
> 解释：在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4。
> 最大总利润为 4 。
> ```
>
> **示例 3：**
>
> ```
> 输入：prices = [7,6,4,3,1]
> 输出：0
> 解释：在这种情况下, 交易无法获得正利润，所以不参与交易可以获得最大利润，最大利润为 0。
> ```
>
>  
>
> **提示：**
>
> - `1 <= prices.length <= 3 * 10^4`
> - `0 <= prices[i] <= 10^4`

```cpp
int maxProfit(vector<int>& prices) {
    int n = prices.size();
    vector<int> f(n); // 买入
    vector<int> g(n); // 卖出
    f[0] = -prices[0];
    for (int i = 1; i < n; ++i) {
        f[i] = max(f[i-1], g[i-1] - prices[i]);
        g[i] = max(f[i-1] + prices[i], g[i-1]);
    }
    return g[n-1];
}
```

## [198. 打家劫舍](https://leetcode.cn/problems/house-robber/)

> 你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。
>
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 **不触动警报装置的情况下** ，一夜之内能够偷窃到的最高金额。
>
>  
>
> **示例 1：**
>
> ```
> 输入：[1,2,3,1]
> 输出：4
> 解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
>      偷窃到的最高金额 = 1 + 3 = 4 。
> ```
>
> **示例 2：**
>
> ```
> 输入：[2,7,9,3,1]
> 输出：12
> 解释：偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
>      偷窃到的最高金额 = 2 + 9 + 1 = 12 。
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 100`
> - `0 <= nums[i] <= 400`

```cpp
int rob(vector<int>& nums) {
    int n = nums.size();
    // dp[i][0]偷盗nums[i]的最高金额
    // dp[i][1]不偷nums[i]的最高金额
    vector<vector<int>> dp(n, vector<int>(2));
    dp[0][0] = nums[0], dp[0][1] = 0;

    for (int i = 1; i < n; ++i) {
        dp[i][0] = dp[i-1][1] + nums[i];
        dp[i][1] = max(dp[i-1][0], dp[i-1][1]);
    }

    return max(dp[n-1][0], dp[n-1][1]);
}
```

## [213. 打家劫舍 II](https://leetcode.cn/problems/house-robber-ii/)

> 你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 **围成一圈** ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警** 。
>
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 **在不触动警报装置的情况下** ，今晚能够偷窃到的最高金额。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [2,3,2]
> 输出：3
> 解释：你不能先偷窃 1 号房屋（金额 = 2），然后偷窃 3 号房屋（金额 = 2）, 因为他们是相邻的。
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [1,2,3,1]
> 输出：4
> 解释：你可以先偷窃 1 号房屋（金额 = 1），然后偷窃 3 号房屋（金额 = 3）。
>      偷窃到的最高金额 = 1 + 3 = 4 。
> ```
>
> **示例 3：**
>
> ```
> 输入：nums = [1,2,3]
> 输出：3
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 100`
> - `0 <= nums[i] <= 1000`

```cpp
int rob(vector<int>& nums) {
    int n = nums.size();
    // 偷第一间房屋，不能偷最后一间房屋
    int a = nums[0] + robRange(nums, 2, n-1);
    // 不偷第一间房屋
    int b = robRange(nums, 1, n);
    return max(a, b);
}

// 在[left, right)内偷取房屋的最大金额
int robRange(vector<int>& nums, int left, int right) {
    if (left >= right) {
        return 0;
    }
    // dp[i][0]偷盗nums[i]的最高金额
    // dp[i][1]不偷nums[i]的最高金额
    vector<vector<int>> dp(right, vector<int>(2));
    dp[left][0] = nums[left], dp[left][1] = 0;

    for (int i = left + 1; i < right; ++i) {
        dp[i][0] = dp[i-1][1] + nums[i];
        dp[i][1] = max(dp[i-1][0], dp[i-1][1]);
    }
    return max(dp[right-1][0], dp[right-1][1]);
}
```

## [309. 买卖股票的最佳时机含冷冻期](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

> 给定一个整数数组`prices`，其中第 `prices[i]` 表示第 `*i*` 天的股票价格 。
>
> 设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:
>
> - 卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。
>
> **注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
>
>  
>
> **示例 1:**
>
> ```
> 输入: prices = [1,2,3,0,2]
> 输出: 3 
> 解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]
> ```
>
> **示例 2:**
>
> ```
> 输入: prices = [1]
> 输出: 0
> ```
>
>  
>
> **提示：**
>
> - `1 <= prices.length <= 5000`
> - `0 <= prices[i] <= 1000`

```cpp
int maxProfit(vector<int>& prices) {
    int n = prices.size();
    vector<vector<int>> dp(n, vector<int>(3));
    // 0->买入状态, 1->可交易状态, 2->冷冻期
    dp[0][0] = -prices[0];
    for (int i = 1; i < n; ++i) {
        dp[i][0] = max(dp[i-1][0], dp[i-1][1] - prices[i]);
        dp[i][1] = max(dp[i-1][1], dp[i-1][2]);
        dp[i][2] = dp[i-1][0] + prices[i];
    }
    return max(dp[n - 1][1], dp[n - 1][2]);
}
```

## [714. 买卖股票的最佳时机含手续费](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

> 给定一个整数数组 `prices`，其中 `prices[i]`表示第 `i` 天的股票价格 ；整数 `fee` 代表了交易股票的手续费用。
>
> 你可以无限次地完成交易，但是你每笔交易都需要付手续费。如果你已经购买了一个股票，在卖出它之前你就不能再继续购买股票了。
>
> 返回获得利润的最大值。
>
> **注意：**这里的一笔交易指买入持有并卖出股票的整个过程，每笔交易你只需要为支付一次手续费。
>
>  
>
> **示例 1：**
>
> ```
> 输入：prices = [1, 3, 2, 8, 4, 9], fee = 2
> 输出：8
> 解释：能够达到的最大利润:  
> 在此处买入 prices[0] = 1
> 在此处卖出 prices[3] = 8
> 在此处买入 prices[4] = 4
> 在此处卖出 prices[5] = 9
> 总利润: ((8 - 1) - 2) + ((9 - 4) - 2) = 8
> ```
>
> **示例 2：**
>
> ```
> 输入：prices = [1,3,7,5,10,3], fee = 3
> 输出：6
> ```
>
>  
>
> **提示：**
>
> - `1 <= prices.length <= 5 * 10^4`
> - `1 <= prices[i] < 5 * 10^4`
> - `0 <= fee < 5 * 10^4`

```cpp
int maxProfit(vector<int>& prices, int fee) {
    int n = prices.size();
    vector<int> f(n); // 买入
    vector<int> g(n); // 卖出
    f[0] = -prices[0];
    for (int i = 1; i < n; ++i) {
        f[i] = max(f[i-1], g[i-1] - prices[i]);
        g[i] = max(f[i-1] + prices[i] - fee, g[i-1]);
    }
    return g[n - 1];
}
```

## [740. 删除并获得点数](https://leetcode.cn/problems/delete-and-earn/)

> 给你一个整数数组 `nums` ，你可以对它进行一些操作。
>
> 每次操作中，选择任意一个 `nums[i]` ，删除它并获得 `nums[i]` 的点数。之后，你必须删除 **所有** 等于 `nums[i] - 1` 和 `nums[i] + 1` 的元素。
>
> 开始你拥有 `0` 个点数。返回你能通过这些操作获得的最大点数。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [3,4,2]
> 输出：6
> 解释：
> 删除 4 获得 4 个点数，因此 3 也被删除。
> 之后，删除 2 获得 2 个点数。总共获得 6 个点数。
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [2,2,3,3,3,4]
> 输出：9
> 解释：
> 删除 3 获得 3 个点数，接着要删除两个 2 和 4 。
> 之后，再次删除 3 获得 3 个点数，再次删除 3 获得 3 个点数。
> 总共获得 9 个点数。
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 2 * 10^4`
> - `1 <= nums[i] <= 10^4`

```cpp
int deleteAndEarn(vector<int>& nums) {
    const int N = 1e4 + 5;
    vector<int> arr(N);
    for (int i = 0; i < nums.size(); ++i) {
        arr[nums[i]] += nums[i];
    }

    // f[i]不选i的最大点数
    // g[i]  选i的最大点数
    vector<int> f(N);
    vector<int> g(N);
    for (int i = 1; i < N; ++i) {
        f[i] = max(f[i-1], g[i-1]);
        g[i] = f[i-1] + arr[i];
    }
    return max(f[N-1], g[N-1]);
}
```

## [LCR 091. 粉刷房子](https://leetcode.cn/problems/JEj789/)

> 假如有一排房子，共 `n` 个，每个房子可以被粉刷成红色、蓝色或者绿色这三种颜色中的一种，你需要粉刷所有的房子并且使其相邻的两个房子颜色不能相同。
>
> 当然，因为市场上不同颜色油漆的价格不同，所以房子粉刷成不同颜色的花费成本也是不同的。每个房子粉刷成不同颜色的花费是以一个 `n x 3` 的正整数矩阵 `costs` 来表示的。
>
> 例如，`costs[0][0]` 表示第 0 号房子粉刷成红色的成本花费；`costs[1][2]` 表示第 1 号房子粉刷成绿色的花费，以此类推。
>
> 请计算出粉刷完所有房子最少的花费成本。
>
>  
>
> **示例 1：**
>
> ```
> 输入: costs = [[17,2,17],[16,16,5],[14,3,19]]
> 输出: 10
> 解释: 将 0 号房子粉刷成蓝色，1 号房子粉刷成绿色，2 号房子粉刷成蓝色。
>      最少花费: 2 + 5 + 3 = 10。
> ```
>
> **示例 2：**
>
> ```
> 输入: costs = [[7,6,2]]
> 输出: 2
> ```
>
>  
>
> **提示:**
>
> - `costs.length == n`
> - `costs[i].length == 3`
> - `1 <= n <= 100`
> - `1 <= costs[i][j] <= 20`
>
>  
>
> 注意：本题与主站 256 题相同：https://leetcode-cn.com/problems/paint-house/

```cpp
int minCost(vector<vector<int>>& costs) {
    int n = costs.size();
    // cost[i-1][0] 粉刷成红色的最少的花费
    vector<vector<int>> dp(n + 1, vector<int>(3));
    for (int i = 1; i <= n; ++i) {
        dp[i][0] = min(dp[i - 1][1], dp[i - 1][2]) + costs[i - 1][0];
        dp[i][1] = min(dp[i - 1][0], dp[i - 1][2]) + costs[i - 1][1];
        dp[i][2] = min(dp[i - 1][0], dp[i - 1][1]) + costs[i - 1][2];
    }
    return min(min(dp[n][0], dp[n][1]), dp[n][2]);
}
```

# 困难

## [123. 买卖股票的最佳时机 III](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)

> 给定一个数组，它的第 `i` 个元素是一支给定的股票在第 `i` 天的价格。
>
> 设计一个算法来计算你所能获取的最大利润。你最多可以完成 **两笔** 交易。
>
> **注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
>
>  
>
> **示例 1:**
>
> ```
> 输入：prices = [3,3,5,0,0,3,1,4]
> 输出：6
> 解释：在第 4 天（股票价格 = 0）的时候买入，在第 6 天（股票价格 = 3）的时候卖出，这笔交易所能获得利润 = 3-0 = 3 。
>      随后，在第 7 天（股票价格 = 1）的时候买入，在第 8 天 （股票价格 = 4）的时候卖出，这笔交易所能获得利润 = 4-1 = 3 。
> ```
>
> **示例 2：**
>
> ```
> 输入：prices = [1,2,3,4,5]
> 输出：4
> 解释：在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。   
>      注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。   
>      因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
> ```
>
> **示例 3：**
>
> ```
> 输入：prices = [7,6,4,3,1] 
> 输出：0 
> 解释：在这个情况下, 没有交易完成, 所以最大利润为 0。
> ```
>
> **示例 4：**
>
> ```
> 输入：prices = [1]
> 输出：0
> ```
>
>  
>
> **提示：**
>
> - `1 <= prices.length <= 105`
> - `0 <= prices[i] <= 105`

简单变化一下，就是这题[188. 买卖股票的最佳时机 IV](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/)的解法。

```cpp
int maxProfit(vector<int>& prices) {
    int n = prices.size();
    const int N = -0x3f3f3f;
    vector<vector<int>> f(n, vector<int>(3, N)); // 买
    vector<vector<int>> g(n, vector<int>(3, N)); // 卖
    f[0][0] = -prices[0], g[0][0] = 0;
    for (int i = 1; i < n; ++i) {
        for (int j = 0; j <= 2; ++j) {
            f[i][j] = max(f[i - 1][j], g[i - 1][j] - prices[i]);
            g[i][j] = g[i - 1][j];
            if (j - 1 >= 0) {
                g[i][j] = max(g[i][j], f[i - 1][j - 1] + prices[i]);
            }
        }
    }
    int ret = 0;
    for (int i = 0; i <= 2; ++i) {
        ret = max(ret, g[n - 1][i]);
    }
    return ret;
}
```

**优化**

```cpp
int maxProfit(vector<int>& prices) {
    int n = prices.size();
    const int N = -0x3f3f3f;
    int f1 = -prices[0], f2 = N; // f 买了
    int g0 = 0, g1 = N, g2 = N;  // g 卖了
    for (int i = 1; i < n; ++i) {
        f1 = max(f1, g0 - prices[i]);
        f2 = max(f2, g1 - prices[i]);
        g1 = max(g1, f1 + prices[i]);
        g2 = max(g2, f2 + prices[i]);
    }
    return max({g0, g1, g2});
}
```

## [188. 买卖股票的最佳时机 IV](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/)

> 给你一个整数数组 `prices` 和一个整数 `k` ，其中 `prices[i]` 是某支给定的股票在第 `i` 天的价格。
>
> 设计一个算法来计算你所能获取的最大利润。你最多可以完成 `k` 笔交易。也就是说，你最多可以买 `k` 次，卖 `k` 次。
>
> **注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
>
>  
>
> **示例 1：**
>
> ```
> 输入：k = 2, prices = [2,4,1]
> 输出：2
> 解释：在第 1 天 (股票价格 = 2) 的时候买入，在第 2 天 (股票价格 = 4) 的时候卖出，这笔交易所能获得利润 = 4-2 = 2 。
> ```
>
> **示例 2：**
>
> ```
> 输入：k = 2, prices = [3,2,6,5,0,3]
> 输出：7
> 解释：在第 2 天 (股票价格 = 2) 的时候买入，在第 3 天 (股票价格 = 6) 的时候卖出, 这笔交易所能获得利润 = 6-2 = 4 。
>      随后，在第 5 天 (股票价格 = 0) 的时候买入，在第 6 天 (股票价格 = 3) 的时候卖出, 这笔交易所能获得利润 = 3-0 = 3 。
> ```
>
>  
>
> **提示：**
>
> - `1 <= k <= 100`
> - `1 <= prices.length <= 1000`
> - `0 <= prices[i] <= 1000`

```cpp
int maxProfit(int k, vector<int>& prices) {
    int n = prices.size();
    const int N = -0x3f3f3f;
    k = min(k, n / 2);
    vector<vector<int>> f(n, vector<int>(k + 1, N)); // 买(i+1)个
    vector<vector<int>> g(n, vector<int>(k + 1, N)); // 卖j个
    f[0][0] = -prices[0], g[0][0] = 0;
    for (int i = 1; i < n; ++i) {
        for (int j = 0; j <= k; ++j) {
            // f[i][j] 买了j+1个
            f[i][j] = max(f[i - 1][j], g[i - 1][j] - prices[i]);
            // g[i][j] 卖了j个
            g[i][j] = g[i - 1][j];
            if (j - 1 >= 0) {
                g[i][j] = max(g[i][j], f[i - 1][j - 1] + prices[i]);
            }
        }
    }
    int ans = 0;
    for (int i = 0; i <= k; ++i) {
        ans = max(ans, g[n - 1][i]);
    }
    return ans;
}
```

