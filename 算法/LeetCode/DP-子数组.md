# 中等

## [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

> 给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
>
> **子数组**是数组中的一个连续部分。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
> 输出：6
> 解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [1]
> 输出：1
> ```
>
> **示例 3：**
>
> ```
> 输入：nums = [5,4,-1,7,8]
> 输出：23
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 10^5`
> - `-10^4 <= nums[i] <= 10^4`
>
> 
>
> **进阶：**如果你已经实现复杂度为 `O(n)` 的解法，尝试使用更为精妙的 **分治法** 求解。

```cpp
int maxSubArray(vector<int>& nums) {
    int n = nums.size();
    // i结尾最大子数组和
    vector<int> dp(n);
    dp[0] = nums[0];
    int ans = dp[0];
    for (int i = 1; i < n; ++i) {
        dp[i] = max(dp[i - 1] + nums[i], nums[i]);
        ans = max(ans, dp[i]);
    }
    return ans;
}
```

## [152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/)

> 给你一个整数数组 `nums` ，请你找出数组中乘积最大的非空连续 子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。
>
> 测试用例的答案是一个 **32-位** 整数。
>
>  
>
> **示例 1:**
>
> ```
> 输入: nums = [2,3,-2,4]
> 输出: 6
> 解释: 子数组 [2,3] 有最大乘积 6。
> ```
>
> **示例 2:**
>
> ```
> 输入: nums = [-2,0,-1]
> 输出: 0
> 解释: 结果不能为 2, 因为 [-2,-1] 不是子数组。
> ```
>
>  
>
> **提示:**
>
> - `1 <= nums.length <= 2 * 104`
> - `-10 <= nums[i] <= 10`
> - `nums` 的任何子数组的乘积都 **保证** 是一个 **32-位** 整数

```cpp
int maxProduct(vector<int>& nums) {
    int n = nums.size();
    vector<int> f(n); // i结尾最大乘积
    vector<int> g(n); // i结尾最小乘积
    f[0] = g[0] = nums[0];
    int ans = f[0];
    for (int i = 1; i < n; ++i) {
        int a = nums[i], b = f[i-1]*a, c = g[i-1]*a;
        f[i] = max({a, b, c});
        g[i] = min({a, b, c});
        ans = max(ans, f[i]);
    }
    return ans;
}
```

## [918. 环形子数组的最大和](https://leetcode.cn/problems/maximum-sum-circular-subarray/)

> 给定一个长度为 `n` 的**环形整数数组** `nums` ，返回 *`nums` 的非空 **子数组** 的最大可能和* 。
>
> **环形数组** 意味着数组的末端将会与开头相连呈环状。形式上， `nums[i]` 的下一个元素是 `nums[(i + 1) % n]` ， `nums[i]` 的前一个元素是 `nums[(i - 1 + n) % n]` 。
>
> **子数组** 最多只能包含固定缓冲区 `nums` 中的每个元素一次。形式上，对于子数组 `nums[i], nums[i + 1], ..., nums[j]` ，不存在 `i <= k1, k2 <= j` 其中 `k1 % n == k2 % n` 。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [1,-2,3,-2]
> 输出：3
> 解释：从子数组 [3] 得到最大和 3
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [5,-3,5]
> 输出：10
> 解释：从子数组 [5,5] 得到最大和 5 + 5 = 10
> ```
>
> **示例 3：**
>
> ```
> 输入：nums = [3,-2,2,-3]
> 输出：3
> 解释：从子数组 [3] 和 [3,-2,2] 都可以得到最大和 3
> ```
>
>  
>
> **提示：**
>
> - `n == nums.length`
> - `1 <= n <= 3 * 10^4`
> - `-3 * 10^4 <= nums[i] <= 3 * 10^4`

```cpp
int maxSubarraySumCircular(vector<int>& nums) {
    int n = nums.size();
    vector<int> f(n); // i结尾最大子数组和
    vector<int> g(n); // i结尾最小子数组和，解决环形最大问题
    f[0] = nums[0], g[0] = nums[0];
    int fmax = f[0], gmin = g[0];
    for (int i = 1; i < n; ++i) {
        f[i] = max(f[i-1] + nums[i], nums[i]);
        g[i] = min(g[i-1] + nums[i], nums[i]);
        fmax = max(fmax, f[i]);
        gmin = min(gmin, g[i]);
    }
    // 所有都是负数
    int sum = accumulate(nums.begin(), nums.end(), 0);
    if (gmin == sum) {
        return fmax;
    }
    return max(sum - gmin, fmax);
}
```

## [1567. 乘积为正数的最长子数组长度](https://leetcode.cn/problems/maximum-length-of-subarray-with-positive-product/)

> 给你一个整数数组 `nums` ，请你求出乘积为正数的最长子数组的长度。
>
> 一个数组的子数组是由原数组中零个或者更多个连续数字组成的数组。
>
> 请你返回乘积为正数的最长子数组长度。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [1,-2,-3,4]
> 输出：4
> 解释：数组本身乘积就是正数，值为 24 。
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [0,1,-2,-3,-4]
> 输出：3
> 解释：最长乘积为正数的子数组为 [1,-2,-3] ，乘积为 6 。
> 注意，我们不能把 0 也包括到子数组中，因为这样乘积为 0 ，不是正数。
> ```
>
> **示例 3：**
>
> ```
> 输入：nums = [-1,-2,-3,0,1]
> 输出：2
> 解释：乘积为正数的最长子数组是 [-1,-2] 或者 [-2,-3] 。
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 10^5`
> - `-10^9 <= nums[i] <= 10^9`

```cpp
int getMaxLen(vector<int>& nums) {
    int n = nums.size();
    vector<int> f(n); // i结尾，乘积为正数的最长长度
    vector<int> g(n); // i结尾，乘积为负数的最长长度
    f[0] = nums[0] > 0, g[0] = nums[0] < 0;
    int ans = f[0];
    for (int i = 1; i < n; ++i) {
        if (nums[i] > 0) {
            f[i] = f[i-1] + 1;
            g[i] = g[i-1] == 0 ? 0 : g[i-1] + 1;
        }
        if (nums[i] < 0) {
            f[i] = g[i-1] == 0 ? 0 : g[i-1] + 1;
            g[i] = f[i-1] + 1;
        }
        ans = max(ans, f[i]);
    }
    return ans;
}
```

