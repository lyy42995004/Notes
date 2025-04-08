# 简单

## [1863. 找出所有子集的异或总和再求和](https://leetcode.cn/problems/sum-of-all-subset-xor-totals/)

> 一个数组的 **异或总和** 定义为数组中所有元素按位 `XOR` 的结果；如果数组为 **空** ，则异或总和为 `0` 。
>
> - 例如，数组 `[2,5,6]` 的 **异或总和** 为 `2 XOR 5 XOR 6 = 1` 。
>
> 给你一个数组 `nums` ，请你求出 `nums` 中每个 **子集** 的 **异或总和** ，计算并返回这些值相加之 **和** 。
>
> **注意：**在本题中，元素 **相同** 的不同子集应 **多次** 计数。
>
> 数组 `a` 是数组 `b` 的一个 **子集** 的前提条件是：从 `b` 删除几个（也可能不删除）元素能够得到 `a` 。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [1,3]
> 输出：6
> 解释：[1,3] 共有 4 个子集：
> - 空子集的异或总和是 0 。
> - [1] 的异或总和为 1 。
> - [3] 的异或总和为 3 。
> - [1,3] 的异或总和为 1 XOR 3 = 2 。
> 0 + 1 + 3 + 2 = 6
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [5,1,6]
> 输出：28
> 解释：[5,1,6] 共有 8 个子集：
> - 空子集的异或总和是 0 。
> - [5] 的异或总和为 5 。
> - [1] 的异或总和为 1 。
> - [6] 的异或总和为 6 。
> - [5,1] 的异或总和为 5 XOR 1 = 4 。
> - [5,6] 的异或总和为 5 XOR 6 = 3 。
> - [1,6] 的异或总和为 1 XOR 6 = 7 。
> - [5,1,6] 的异或总和为 5 XOR 1 XOR 6 = 2 。
> 0 + 5 + 1 + 6 + 4 + 3 + 7 + 2 = 28
> ```
>
> **示例 3：**
>
> ```
> 输入：nums = [3,4,5,6,7,8]
> 输出：480
> 解释：每个子集的全部异或总和值之和为 480 。
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 12`
> - `1 <= nums[i] <= 20`

```cpp
int ans = 0;
int path = 0;

int subsetXORSum(vector<int>& nums) {
    dfs(nums, 0);
    return ans;
}

void dfs(vector<int>& nums, int pos) {
    ans += path;
    for (int i = pos; i < nums.size(); ++i) {
        path ^= nums[i];
        dfs(nums, i + 1);
        path ^= nums[i];
    }
}
```

## [面试题 08.06. 汉诺塔问题](https://leetcode.cn/problems/hanota-lcci/)

> 在经典汉诺塔问题中，有 3 根柱子及 N 个不同大小的穿孔圆盘，盘子可以滑入任意一根柱子。一开始，所有盘子自上而下按升序依次套在第一根柱子上(即每一个盘子只能放在更大的盘子上面)。移动圆盘时受到以下限制:
> (1) 每次只能移动一个盘子;
> (2) 盘子只能从柱子顶端滑出移到下一根柱子;
> (3) 盘子只能叠在比它大的盘子上。
>
> 请编写程序，用栈将所有盘子从第一根柱子移到最后一根柱子。
>
> 你需要原地修改栈。
>
> **示例 1：**
>
> ```
>  输入：A = [2, 1, 0], B = [], C = []
>  输出：C = [2, 1, 0]
> ```
>
> **示例 2：**
>
> ```
>  输入：A = [1, 0], B = [], C = []
>  输出：C = [1, 0]
> ```
>
> **提示：**
>
> 1. A 中盘子的数目不大于 14 个。

```cpp
void hanota(vector<int>& A, vector<int>& B, vector<int>& C) {
    move(A, B, C, A.size());
}

void move(vector<int>& A, vector<int>& B, vector<int>& C, int n) {
    if (n == 1) {
        C.push_back(A.back());
        A.pop_back();
        return;
    }

    move(A, C, B, n - 1); // 将A上的n-1个借助C移到B上
    C.push_back(A.back());
    A.pop_back();
    move(B, A, C, n - 1); // 将B上的n-1个借助A移到C上
}
```

# 中等

## [50. Pow(x, n)](https://leetcode.cn/problems/powx-n/)

> 实现 [pow(*x*, *n*)](https://www.cplusplus.com/reference/valarray/pow/) ，即计算 `x` 的整数 `n` 次幂函数（即，`xn` ）。
>
>  
>
> **示例 1：**
>
> ```
> 输入：x = 2.00000, n = 10
> 输出：1024.00000
> ```
>
> **示例 2：**
>
> ```
> 输入：x = 2.10000, n = 3
> 输出：9.26100
> ```
>
> **示例 3：**
>
> ```
> 输入：x = 2.00000, n = -2
> 输出：0.25000
> 解释：2-2 = 1/22 = 1/4 = 0.25
> ```
>
>  
>
> **提示：**
>
> - `-100.0 < x < 100.0`
> - `-2^31 <= n <= 2^31-1`
> - `n` 是一个整数
> - 要么 `x` 不为零，要么 `n > 0` 。
> - `-10^4 <= x^n <= 10^4`

```cpp
double myPow(double x, int n) {
    return n >= 0 ? quickMul(x, n) : 1.0 / quickMul(x, -(long long)n);
}

double quickMul(double x, long long n) {
    if (n == 0) {
        return 1.0;
    }
    double tmp = quickMul(x, n / 2);
    return (n % 2 ? tmp * tmp * x : tmp * tmp);
}
```

## [46. 全排列](https://leetcode.cn/problems/permutations/)

> 给定一个不含重复数字的数组 `nums` ，返回其 *所有可能的全排列* 。你可以 **按任意顺序** 返回答案。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [1,2,3]
> 输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [0,1]
> 输出：[[0,1],[1,0]]
> ```
>
> **示例 3：**
>
> ```
> 输入：nums = [1]
> 输出：[[1]]
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 6`
> - `-10 <= nums[i] <= 10`
> - `nums` 中的所有整数 **互不相同**

```cpp
vector<vector<int>> ans;
vector<int> path;
vector<bool> vis;

vector<vector<int>> permute(vector<int>& nums) {
    int n = nums.size();
    vis = vector<bool>(n, false);
    dfs(nums);
    return ans;
}

void dfs(vector<int>& nums) {
    int n = nums.size();
    if (path.size() == n) {
        ans.push_back(path);
        return;
    }
    for (int i = 0; i < n; ++i) {
        if (vis[i] == false) {
            path.push_back(nums[i]);
            vis[i] = true;
            dfs(nums);
            path.pop_back();
            vis[i] = false;
        }
    }
}
```

## [47. 全排列 II](https://leetcode.cn/problems/permutations-ii/)

>  给定一个可包含重复数字的序列 `nums` ，***按任意顺序*** 返回所有不重复的全排列。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [1,1,2]
> 输出：
> [[1,1,2],
>  [1,2,1],
>  [2,1,1]]
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [1,2,3]
> 输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 8`
> - `-10 <= nums[i] <= 10`

```cpp
vector<vector<int>> ans;
vector<int> path;
vector<bool> vis;

vector<vector<int>> permuteUnique(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vis = vector<bool>(nums.size(), false);
    dfs(nums, 0);
    return ans;
}

void dfs(vector<int>& nums, int pos) {
    if (path.size() == nums.size()) {
        ans.push_back(path);
        return;
    }
    for (int i = 0; i < nums.size(); ++i) {
        if (vis[i] == true || (i > 0 && nums[i - 1] == nums[i] && vis[i - 1] == true)) {
            continue;
        }
        path.push_back(nums[i]);
        vis[i] = true;
        dfs(nums, i + 1);
        vis[i] = false;
        path.pop_back();
    }
}
```

## [78. 子集](https://leetcode.cn/problems/subsets/)

> 给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。
>
> 解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums = [1,2,3]
> 输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
> ```
>
> **示例 2：**
>
> ```
> 输入：nums = [0]
> 输出：[[],[0]]
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums.length <= 10`
> - `-10 <= nums[i] <= 10`
> - `nums` 中的所有元素 **互不相同**

```cpp
vector<vector<int>> ans;
vector<int> path;

vector<vector<int>> subsets(vector<int>& nums) {
    dfs(nums, 0);
    return ans;
}

void dfs(vector<int>& nums, int pos) {
    if (pos == nums.size()) {
        ans.push_back(path);
        return;
    }
    dfs(nums, pos + 1);

    path.push_back(nums[pos]);
    dfs(nums, pos + 1);
    path.pop_back();
}
```

另一种方法

```cpp
vector<vector<int>> ans;
vector<int> path;

vector<vector<int>> subsets(vector<int>& nums) {
    dfs(nums, 0);
    return ans;
}

void dfs(vector<int>& nums, int pos) {
    ans.push_back(path);
    for (int i = pos; i < nums.size(); ++i) {
        path.push_back(nums[i]);
        dfs(nums, i + 1);
        path.pop_back();
    }
}
```

