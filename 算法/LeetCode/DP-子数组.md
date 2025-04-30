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

## [139. 单词拆分](https://leetcode.cn/problems/word-break/)

> 给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。
>
> **注意：**不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。
>
>  
>
> **示例 1：**
>
> ```
> 输入: s = "leetcode", wordDict = ["leet", "code"]
> 输出: true
> 解释: 返回 true 因为 "leetcode" 可以由 "leet" 和 "code" 拼接成。
> ```
>
> **示例 2：**
>
> ```
> 输入: s = "applepenapple", wordDict = ["apple", "pen"]
> 输出: true
> 解释: 返回 true 因为 "applepenapple" 可以由 "apple" "pen" "apple" 拼接成。
>      注意，你可以重复使用字典中的单词。
> ```
>
> **示例 3：**
>
> ```
> 输入: s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
> 输出: false
> ```
>
>  
>
> **提示：**
>
> - `1 <= s.length <= 300`
> - `1 <= wordDict.length <= 1000`
> - `1 <= wordDict[i].length <= 20`
> - `s` 和 `wordDict[i]` 仅由小写英文字母组成
> - `wordDict` 中的所有字符串 **互不相同**

```cpp
bool wordBreak(string s, vector<string>& wordDict) {
    unordered_set<string> hash;
    for (string str : wordDict) {
        hash.insert(str);
    }

    int n = s.size();
    s = " " + s;
    // i结尾的字符串是否可以拼接出
    vector<bool> dp(n+1);
    dp[0] = true;
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= i; ++j) {
            if (dp[j-1] && hash.count(s.substr(j, i-j+1))) {
                dp[i] = true;
                break;
            }
        }
    }
    return dp[n];
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

## [413. 等差数列划分](https://leetcode.cn/problems/arithmetic-slices/)

> 如果一个数列 **至少有三个元素** ，并且任意两个相邻元素之差相同，则称该数列为等差数列。
>
> - 例如，`[1,3,5,7,9]`、`[7,7,7,7]` 和 `[3,-1,-5,-9]` 都是等差数列。
>
> 给你一个整数数组 `nums` ，返回数组 `nums` 中所有为等差数组的 **子数组** 个数。
>
> **子数组** 是数组中的一个连续序列。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [1,2,3,4]
> 输出：3
> 解释：nums 中有三个子等差数组：[1, 2, 3]、[2, 3, 4] 和 [1,2,3,4] 自身。
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [1]
> 输出：0
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 5000`
> - `-1000 <= nums[i] <= 1000`

```cpp
int numberOfArithmeticSlices(vector<int>& nums) {
    int n = nums.size();
    // i结尾等差子数组个数
    vector<int> dp(n);
    int ans = 0;
    for (int i = 2; i < n; ++i) {
        if (nums[i-2] + nums[i] == 2 * nums[i-1]) {
            dp[i] = dp[i-1] + 1;
        }
        ans += dp[i];
    }
    return ans;
}
```

## [467. 环绕字符串中唯一的子字符串](https://leetcode.cn/problems/unique-substrings-in-wraparound-string/)

>  定义字符串 `base` 为一个 `"abcdefghijklmnopqrstuvwxyz"` 无限环绕的字符串，所以 `base` 看起来是这样的：
>
> - `"...zabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcd...."`.
>
> 给你一个字符串 `s` ，请你统计并返回 `s` 中有多少 **不同****非空子串** 也在 `base` 中出现。
>
>  
>
> **示例 1：**
>
> ```
> 输入：s = "a"
> 输出：1
> 解释：字符串 s 的子字符串 "a" 在 base 中出现。
> ```
>
> **示例 2：**
>
> ```
> 输入：s = "cac"
> 输出：2
> 解释：字符串 s 有两个子字符串 ("a", "c") 在 base 中出现。
> ```
>
> **示例 3：**
>
> ```
> 输入：s = "zab"
> 输出：6
> 解释：字符串 s 有六个子字符串 ("z", "a", "b", "za", "ab", and "zab") 在 base 中出现。
> ```
>
>  
>
> **提示：**
>
> - `1 <= s.length <= 10^5`
> - s 由小写英文字母组成

```cpp
int findSubstringInWraproundString(string s) {
    int n = s.size();
    // i结尾，不同非空子串在 base 中出现的个数
    vector<int> dp(n, 1);
    vector<int> hash(26);
    hash[s[0]-'a'] = 1;
    for (int i = 1; i < n; ++i) {
        if ((s[i] == s[i-1] + 1) || (s[i] == 'a' && s[i-1] == 'z')) {
            dp[i] = dp[i-1] + 1;
        }
        hash[s[i]-'a'] = max(hash[s[i]-'a'], dp[i]);
    }
    int ans = 0;
    for (int i : hash) {
        ans += i;
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

## [978. 最长湍流子数组](https://leetcode.cn/problems/longest-turbulent-subarray/)

> 给定一个整数数组 `arr` ，返回 `arr` 的 *最大湍流子数组的**长度*** 。
>
> 如果比较符号在子数组中的每个相邻元素对之间翻转，则该子数组是 **湍流子数组** 。
>
> 更正式地来说，当 `arr` 的子数组 `A[i], A[i+1], ..., A[j]` 满足仅满足下列条件时，我们称其为*湍流子数组*：
>
> - 若 `i <= k < j`：
>   - 当 `k` 为奇数时， `A[k] > A[k+1]`，且
>   - 当 `k` 为偶数时，`A[k] < A[k+1]`；
> - 或 若 `i <= k < j`：
>   - 当 `k` 为偶数时，`A[k] > A[k+1]` ，且
>   - 当 `k` 为奇数时， `A[k] < A[k+1]`。
>
>  
>
> **示例 1：**
>
> ```
> 输入：arr = [9,4,2,10,7,8,8,1,9]
> 输出：5
> 解释：arr[1] > arr[2] < arr[3] > arr[4] < arr[5]
> ```
>
> **示例 2：**
>
> ```
> 输入：arr = [4,8,12,16]
> 输出：2
> ```
>
> **示例 3：**
>
> ```
> 输入：arr = [100]
> 输出：1
> ```
>
>  
>
> **提示：**
>
> - `1 <= arr.length <= 4 * 104`
> - `0 <= arr[i] <= 109`

```cpp
int maxTurbulenceSize(vector<int>& arr) {
    int n = arr.size();
    vector<int> f(n, 1); // i结尾，上升趋势的最大湍流子数组
    vector<int> g(n, 1); // 下降
    int ans = 1;
    for (int i = 1; i < n; ++i) {
        int a = arr[i - 1], b = arr[i];
        if (a > b) {
            g[i] = f[i - 1] + 1;
        }
        if (a < b) {
            f[i] = g[i - 1] + 1;
        }
        ans = max({ans, f[i], g[i]});
    }
    return ans;
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

