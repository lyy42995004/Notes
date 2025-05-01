# 中等

## [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

> 给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。
>
> **子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [10,9,2,5,3,7,101,18]
> 输出：4
> 解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [0,1,0,3,2,3]
> 输出：4
> ```
>
> **示例 3：**
>
> ```
> 输入：nums = [7,7,7,7,7,7,7]
> 输出：1
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 2500`
> - `-10^4 <= nums[i] <= 10^4`
>
>  
>
> **进阶：**
>
> - 你能将算法的时间复杂度降低到 `O(n log(n))` 吗?

```cpp
int lengthOfLIS(vector<int>& nums) {
    int ans = 1;
    int n = nums.size();
    // 到nums[i]的最长递增子序列
    vector<int> dp(n, 1);
    dp[0] = 1;
    for (int i = 1; i < n; ++i) {	
        for (int j = i - 1; j >= 0; --j) {
            if (nums[j] < nums[i]) {
                dp[i] = max(dp[i], dp[j] + 1);
            }
        }
        ans = max(ans, dp[i]);
    }
    return ans;
}
```

## [376. 摆动序列](https://leetcode.cn/problems/wiggle-subsequence/)

> 如果连续数字之间的差严格地在正数和负数之间交替，则数字序列称为 **摆动序列 。**第一个差（如果存在的话）可能是正数或负数。仅有一个元素或者含两个不等元素的序列也视作摆动序列。
>
> - 例如， `[1, 7, 4, 9, 2, 5]` 是一个 **摆动序列** ，因为差值 `(6, -3, 5, -7, 3)` 是正负交替出现的。
> - 相反，`[1, 4, 7, 2, 5]` 和 `[1, 7, 4, 5, 5]` 不是摆动序列，第一个序列是因为它的前两个差值都是正数，第二个序列是因为它的最后一个差值为零。
>
> **子序列** 可以通过从原始序列中删除一些（也可以不删除）元素来获得，剩下的元素保持其原始顺序。
>
> 给你一个整数数组 `nums` ，返回 `nums` 中作为 **摆动序列** 的 **最长子序列的长度** 。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [1,7,4,9,2,5]
> 输出：6
> 解释：整个序列均为摆动序列，各元素之间的差值为 (6, -3, 5, -7, 3) 。
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [1,17,5,10,13,15,10,5,16,8]
> 输出：7
> 解释：这个序列包含几个长度为 7 摆动序列。
> 其中一个是 [1, 17, 10, 13, 10, 16, 8] ，各元素之间的差值为 (16, -7, 3, -3, 6, -8) 。
> ```
>
> **示例 3：**
>
> ```
> 输入：nums = [1,2,3,4,5,6,7,8,9]
> 输出：2
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 1000`
> - `0 <= nums[i] <= 1000`
>
>  
>
> **进阶：**你能否用 `O(n)` 时间复杂度完成此题?

```cpp
int wiggleMaxLength(vector<int>& nums) {
    int n = nums.size();
    vector<int> f(n, 1); // 上升
    vector<int> g(n, 1); // 下降
    for (int i = 1; i < n; ++i) {
        for (int j = 0; j < i; ++j) {
            if (nums[i] > nums[j]) {
                f[i] = max(f[i], g[j] + 1);
            }
            if (nums[i] < nums[j]) {
                g[i] = max(g[i], f[j] + 1);
            }
        }
    }
    return max(f[n-1], g[n-1]);
}
```

## [646. 最长数对链](https://leetcode.cn/problems/maximum-length-of-pair-chain/)

> 给你一个由 `n` 个数对组成的数对数组 `pairs` ，其中 `pairs[i] = [lefti, righti]` 且 `lefti < righti` 。
>
> 现在，我们定义一种 **跟随** 关系，当且仅当 `b < c` 时，数对 `p2 = [c, d]` 才可以跟在 `p1 = [a, b]` 后面。我们用这种形式来构造 **数对链** 。
>
> 找出并返回能够形成的 **最长数对链的长度** 。
>
> 你不需要用到所有的数对，你可以以任何顺序选择其中的一些数对来构造。
>
>  
>
> **示例 1：**
>
> ```
> 输入：pairs = [[1,2], [2,3], [3,4]]
> 输出：2
> 解释：最长的数对链是 [1,2] -> [3,4] 。
> ```
>
> **示例 2：**
>
> ```
> 输入：pairs = [[1,2],[7,8],[4,5]]
> 输出：3
> 解释：最长的数对链是 [1,2] -> [4,5] -> [7,8] 。
> ```
>
>  
>
> **提示：**
>
> - `n == pairs.length`
> - `1 <= n <= 1000`
> - `-1000 <= lefti < righti <= 1000`

```cpp
int findLongestChain(vector<vector<int>>& pairs) {
    sort(pairs.begin(), pairs.end());
    int n = pairs.size();
    // i结尾，最长数对链的长度
    vector<int> dp(n, 1);
    int ans = 1;
    for (int i = 1; i < n; ++i) {
        for (int j = 0; j < i; ++j) {
            if (pairs[j][1] < pairs[i][0])
                dp[i] = max(dp[i], dp[j] + 1);
        }
        ans = max(ans, dp[i]);
    }
    return ans;
}
```

## [673. 最长递增子序列的个数](https://leetcode.cn/problems/number-of-longest-increasing-subsequence/)

> 给定一个未排序的整数数组 `nums` ， *返回最长递增子序列的个数* 。
>
> **注意** 这个数列必须是 **严格** 递增的。
>
>  
>
> **示例 1:**
>
> ```
> 输入: [1,3,5,4,7]
> 输出: 2
> 解释: 有两个最长递增子序列，分别是 [1, 3, 4, 7] 和[1, 3, 5, 7]。
> ```
>
> **示例 2:**
>
> ```
> 输入: [2,2,2,2,2]
> 输出: 5
> 解释: 最长递增子序列的长度是1，并且存在5个子序列的长度为1，因此输出5。
> ```
>
>  
>
> **提示:** 
>
> - `1 <= nums.length <= 2000`
> - `-10^6 <= nums[i] <= 10^6`

```cpp
int findNumberOfLIS(vector<int>& nums) {
    int n = nums.size();
    vector<int> len(n, 1); // i结尾，最长递增子序列的长度
    vector<int> cnt(n, 1); // i结尾，最长递增子序列的数量
    int ans = 1, maxlen = 1;
    for (int i = 1; i < n; ++i) {
        for (int j = 0; j < i; ++j) {
            if (nums[i] > nums[j]) {
                if (len[j] + 1 == len[i]) {
                    cnt[i] += cnt[j];
                }
                if (len[j] + 1 > len[i]) {
                    cnt[i] = cnt[j];
                    len[i] = len[j] + 1;
                }
            }
        }
        if (len[i] == maxlen) {
            ans += cnt[i];
        }
        if (len[i] > maxlen) {
            maxlen = len[i]; 
            ans = cnt[i];
        }
    }
    return ans;
}
```

## [873. 最长的斐波那契子序列的长度](https://leetcode.cn/problems/length-of-longest-fibonacci-subsequence/)

> 如果序列 `X_1, X_2, ..., X_n` 满足下列条件，就说它是 *斐波那契式* 的：
>
> - `n >= 3`
> - 对于所有 `i + 2 <= n`，都有 `X_i + X_{i+1} = X_{i+2}`
>
> 给定一个**严格递增**的正整数数组形成序列 arr ，找到 arr 中最长的斐波那契式的子序列的长度。如果一个不存在，返回 0 。
>
> *（回想一下，子序列是从原序列 arr 中派生出来的，它从 arr 中删掉任意数量的元素（也可以不删），而不改变其余元素的顺序。例如， `[3, 5, 8]` 是 `[3, 4, 5, 6, 7, 8]` 的一个子序列）*
>
>  
>
> **示例 1：**
>
> ```
> 输入: arr = [1,2,3,4,5,6,7,8]
> 输出: 5
> 解释: 最长的斐波那契式子序列为 [1,2,3,5,8] 。
> ```
>
> **示例 2：**
>
> ```
> 输入: arr = [1,3,7,11,12,14,18]
> 输出: 3
> 解释: 最长的斐波那契式子序列有 [1,11,12]、[3,11,14] 以及 [7,11,18] 。
> ```
>
>  
>
> **提示：**
>
> - `3 <= arr.length <= 1000`
> - `1 <= arr[i] < arr[i + 1] <= 10^9`

```cpp
int lenLongestFibSubseq(vector<int>& arr) {
    int n = arr.size();
    // 存储元素与下标的映射
    unordered_map<int, int> hash;
    for (int i = 0; i < n; ++i)
        hash[arr[i]] = i;
    // arr[i] 和 arr[j] 作为倒数第二个和最后一个元素的最长斐波序列长度
    vector<vector<int>> dp(n, vector<int>(n, 2));
    int ans = 2;
    for (int j = 2; j < n; ++j) {
        for (int i = 1; i < j; ++i) {
            int num = arr[j] - arr[i];
            if (num < arr[i] && hash.count(num)) {
                dp[i][j] = dp[hash[num]][i] + 1;
            }
            ans = max(dp[i][j], ans);
        }
    }
    return ans > 2 ? ans : 0;
}
```

## [1027. 最长等差数列](https://leetcode.cn/problems/longest-arithmetic-subsequence/)

> 给你一个整数数组 `nums`，返回 `nums` 中最长等差子序列的**长度**。
>
> 回想一下，`nums` 的子序列是一个列表 `nums[i1], nums[i2], ..., nums[ik]` ，且 `0 <= i1 < i2 < ... < ik <= nums.length - 1`。并且如果 `seq[i+1] - seq[i]`( `0 <= i < seq.length - 1`) 的值都相同，那么序列 `seq` 是等差的。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [3,6,9,12]
> 输出：4
> 解释： 
> 整个数组是公差为 3 的等差数列。
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [9,4,7,2,10]
> 输出：3
> 解释：
> 最长的等差子序列是 [4,7,10]。
> ```
>
> **示例 3：**
>
> ```
> 输入：nums = [20,1,15,3,10,5,8]
> 输出：4
> 解释：
> 最长的等差子序列是 [20,15,10,5]。
> ```
>
>  
>
> **提示：**
>
> - `2 <= nums.length <= 1000`
> - `0 <= nums[i] <= 500`

```cpp
int longestArithSeqLength(vector<int>& nums) {
    int n = nums.size();
    unordered_map<int, int> hash;
    hash[nums[0]] = 0;
    int ans = 2;
    // arr[i] 和 arr[j] 作为倒数第二个和最后一个元素的最长等差数列
    vector<vector<int>> dp(n, vector<int>(n, 2));
    for (int i = 1; i < n; ++i) {
        for (int j = i + 1; j < n; ++j) {
            int num = 2 * nums[i] - nums[j];
            if (hash.count(num)) {
                dp[i][j] = dp[hash[num]][i] + 1;
            }
            ans = max(ans, dp[i][j]);
        }
        hash[nums[i]] = i; // 每次将判断完的nums[i]对应的下标i存到hash中
    }
    return ans;
}
```

## [1218. 最长定差子序列](https://leetcode.cn/problems/longest-arithmetic-subsequence-of-given-difference/)

> 给你一个整数数组 `arr` 和一个整数 `difference`，请你找出并返回 `arr` 中最长等差子序列的长度，该子序列中相邻元素之间的差等于 `difference` 。
>
> **子序列** 是指在不改变其余元素顺序的情况下，通过删除一些元素或不删除任何元素而从 `arr` 派生出来的序列。
>
>  
>
> **示例 1：**
>
> ```
> 输入：arr = [1,2,3,4], difference = 1
> 输出：4
> 解释：最长的等差子序列是 [1,2,3,4]。
> ```
>
> **示例 2：**
>
> ```
> 输入：arr = [1,3,5,7], difference = 1
> 输出：1
> 解释：最长的等差子序列是任意单个元素。
> ```
>
> **示例 3：**
>
> ```
> 输入：arr = [1,5,7,8,5,3,4,2,1], difference = -2
> 输出：4
> 解释：最长的等差子序列是 [7,5,3,1]。
> ```
>
>  
>
> **提示：**
>
> - `1 <= arr.length <= 10^5`
> - `-10^4 <= arr[i], difference <= 10^4`

```cpp
int longestSubsequence(vector<int>& arr, int difference) {
    // 以 arr[i] 结尾的最长定差子序列长度
    unordered_map<int, int> dp;
    int ans = 0;
    for (int arri : arr) {
        dp[arri] = dp[arri - difference] + 1;
        ans = max(ans, dp[arri]);
    }
    return ans;
}
```

# 困难

## [446. 等差数列划分 II - 子序列](https://leetcode.cn/problems/arithmetic-slices-ii-subsequence/)

> 给你一个整数数组 `nums` ，返回 `nums` 中所有 **等差子序列** 的数目。
>
> 如果一个序列中 **至少有三个元素** ，并且任意两个相邻元素之差相同，则称该序列为等差序列。
>
> - 例如，`[1, 3, 5, 7, 9]`、`[7, 7, 7, 7]` 和 `[3, -1, -5, -9]` 都是等差序列。
> - 再例如，`[1, 1, 2, 5, 7]` 不是等差序列。
>
> 数组中的子序列是从数组中删除一些元素（也可能不删除）得到的一个序列。
>
> - 例如，`[2,5,10]` 是 `[1,2,1,2,4,1,5,10]` 的一个子序列。
>
> 题目数据保证答案是一个 **32-bit** 整数。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [2,4,6,8,10]
> 输出：7
> 解释：所有的等差子序列为：
> [2,4,6]
> [4,6,8]
> [6,8,10]
> [2,4,6,8]
> [4,6,8,10]
> [2,4,6,8,10]
> [2,6,10]
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [7,7,7,7,7]
> 输出：16
> 解释：数组中的任意子序列都是等差子序列。
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 1000`
> - `-2^31 <= nums[i] <= 2^31 - 1`

```cpp
int numberOfArithmeticSlices(vector<int>& nums) {
    int n = nums.size();
    unordered_map<long long, vector<int>> hash;
    for (int i = 0; i < n; ++i)
        hash[nums[i]].push_back(i);
    // nums[i] 和 nums[j] 作为倒数第二个和最后一个元素的等差子序列的数量
    vector<vector<int>> dp(n, vector<int>(n));
    int ans = 0;
    for (int i = 1; i < n; ++i) {
        for (int j = i + 1; j < n; ++j) {
            long long num = (long long)2 * nums[i] - nums[j];
            if (hash.count(num)) {
                for (auto k : hash[num]) {
                    if (k < i) {
                        dp[i][j] += dp[k][i] + 1;
                    }
                }
            }
            ans += dp[i][j];
        }
    }
    return ans;
}
```

