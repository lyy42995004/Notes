# 中等

## [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

> 给你一个字符串 `s`，找到 `s` 中最长的 回文 子串。
>
>  
>
> **示例 1：**
>
> ```
> 输入：s = "babad"
> 输出："bab"
> 解释："aba" 同样是符合题意的答案。
> ```
>
> **示例 2：**
>
> ```
> 输入：s = "cbbd"
> 输出："bb"
> ```
>
>  
>
> **提示：**
>
> - `1 <= s.length <= 1000`
> - `s` 仅由数字和英文字母组成

```cpp
string longestPalindrome(string s) {
    int n = s.size();
    // i开始j结尾的子串是否回文
    vector<vector<bool>> dp(n, vector<bool>(n));
    int begin, len = 0;
    for (int i = n - 1; i >= 0; --i) {
        for (int j = i; j < n; ++j) {
            if (s[i] == s[j]) {
                dp[i][j] = j-i+1 > 2 ? dp[i+1][j-1] : true;
            }
            if (dp[i][j] == true && j-i+1 > len) {
                len = j-i+1;
                begin = i;
            }
        }
    }
    return s.substr(begin, len);
}
```

## [647. 回文子串](https://leetcode.cn/problems/palindromic-substrings/)

> 给你一个字符串 `s` ，请你统计并返回这个字符串中 **回文子串** 的数目。
>
> **回文字符串** 是正着读和倒过来读一样的字符串。
>
> **子字符串** 是字符串中的由连续字符组成的一个序列。
>
>  
>
> **示例 1：**
>
> ```
> 输入：s = "abc"
> 输出：3
> 解释：三个回文子串: "a", "b", "c"
> ```
>
> **示例 2：**
>
> ```
> 输入：s = "aaa"
> 输出：6
> 解释：6个回文子串: "a", "a", "a", "aa", "aa", "aaa"
> ```
>
>  
>
> **提示：**
>
> - `1 <= s.length <= 1000`
> - `s` 由小写英文字母组成

```cpp
int countSubstrings(string s) {
    int n = s.size();
    // i开始j结尾的子串是否回文
    vector<vector<bool>> dp(n, vector<bool>(n));
    int ans = 0;
    for (int i = n - 1; i >= 0; --i) {
        for (int j = i; j < n; ++j) {
            if (s[i] == s[j]) {
                dp[i][j] = j-i+1 > 2 ? dp[i+1][j-1] : true;
            }
            if (dp[i][j] == true) {
                ans++;
            }
        }
    }
    return ans;
}
```

# 困难

## [132. 分割回文串 II](https://leetcode.cn/problems/palindrome-partitioning-ii/)

> 给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是回文串。
>
> 返回符合要求的 **最少分割次数** 。
>
>  
>
> **示例 1：**
>
> ```
> 输入：s = "aab"
> 输出：1
> 解释：只需一次分割就可将 s 分割成 ["aa","b"] 这样两个回文子串。
> ```
>
> **示例 2：**
>
> ```
> 输入：s = "a"
> 输出：0
> ```
>
> **示例 3：**
>
> ```
> 输入：s = "ab"
> 输出：1
> ```
>
>  
>
> **提示：**
>
> - `1 <= s.length <= 2000`
> - `s` 仅由小写英文字母组成

```cpp
int minCut(string s) {
    int n = s.size();
    // i开始j结尾的子串是否回文
    vector<vector<bool>> check(n, vector<bool>(n));
    for (int i = n - 1; i >= 0; i--) {
        for (int j = i; j < n; j++) {
            if (s[i] == s[j]) {
                check[i][j] = j-i+1 > 2 ? check[i + 1][j - 1] : true;
            }
        }
    }
    // i结尾的最少分割次数
    vector<int> dp(n, INT_MAX);
    dp[0] = 0;
    for (int i = 1; i < n; ++i) {
        if (check[0][i] == true) {
            dp[i] = 0;
            continue;
        }
        for (int j = 0; j < i; ++j) {
            if (check[j+1][i]) {
                dp[i] = min(dp[i], dp[j] + 1);
            }
        }
    }
    return dp[n - 1];
}
```

## [1278. 分割回文串 III](https://leetcode.cn/problems/palindrome-partitioning-iii/)

> 给你一个由小写字母组成的字符串 `s`，和一个整数 `k`。
>
> 请你按下面的要求分割字符串：
>
> - 首先，你可以将 `s` 中的部分字符修改为其他的小写英文字母。
> - 接着，你需要把 `s` 分割成 `k` 个非空且不相交的子串，并且每个子串都是回文串。
>
> 请返回以这种方式分割字符串所需修改的最少字符数。
>
>  
>
> **示例 1：**
>
> ```
> 输入：s = "abc", k = 2
> 输出：1
> 解释：你可以把字符串分割成 "ab" 和 "c"，并修改 "ab" 中的 1 个字符，将它变成回文串。
> ```
>
> **示例 2：**
>
> ```
> 输入：s = "aabbc", k = 3
> 输出：0
> 解释：你可以把字符串分割成 "aa"、"bb" 和 "c"，它们都是回文串。
> ```
>
> **示例 3：**
>
> ```
> 输入：s = "leetcode", k = 8
> 输出：0
> ```
>
>  
>
> **提示：**
>
> - `1 <= k <= s.length <= 100`
> - `s` 中只含有小写英文字母。

```cpp
int palindromePartition(string s, int k) {
    int n = s.size();
    // i-1结尾，分割为j个，所需修改的最少字符数
    vector<vector<int>> dp(n+1, vector<int>(k+1, INT_MAX));
    dp[0][0] = 0;
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= min(k, i); ++j) {
            if (j == 1) {
                dp[i][j] = cost(s, 0, i - 1);
            } else {
                // z从(j-1+1)-1开始，表示前面的j-1个子串
                for (int z = j - 1; z < i; ++z) {
                    dp[i][j] = min(dp[i][j], dp[z][j-1] + cost(s, z, i-1));
                }
            }
        }
    }
    return dp[n][k];
}

int cost(const string& s, int left, int right) {
    int ret = 0;
    while (left < right) {
        if (s[left] != s[right]) {
            ret++;
        }
        left++, right--;
    }
    return ret;
}
```

## [1745. 分割回文串 IV](https://leetcode.cn/problems/palindrome-partitioning-iv/)

> 给你一个字符串 `s` ，如果可以将它分割成三个 **非空** 回文子字符串，那么返回 `true` ，否则返回 `false` 。
>
> 当一个字符串正着读和反着读是一模一样的，就称其为 **回文字符串** 。
>
>  
>
> **示例 1：**
>
> ```
> 输入：s = "abcbdd"
> 输出：true
> 解释："abcbdd" = "a" + "bcb" + "dd"，三个子字符串都是回文的。
> ```
>
> **示例 2：**
>
> ```
> 输入：s = "bcbddxy"
> 输出：false
> 解释：s 没办法被分割成 3 个回文子字符串。
> ```
>
>  
>
> **提示：**
>
> - `3 <= s.length <= 2000`
> - `s` 只包含小写英文字母。

```cpp
bool checkPartitioning(string s) {
    int n = s.size();
    // i开始j结尾的子串是否回文
    vector<vector<bool>> dp(n, vector<bool>(n));
    for (int i = n - 1; i >= 0; i--) {
        for (int j = i; j < n; j++) {
            if (s[i] == s[j]) {
                dp[i][j] = j-i+1 > 2 ? dp[i + 1][j - 1] : true;
            }
        }
    }
    // 枚举三个区间
    for (int i = 1; i < n - 1; ++i) {
        for (int j = i; j < n - 1; ++j) {
            if (dp[0][i-1] && dp[i][j] && dp[j+1][n-1]) {
                return true;
            }
        }
    }

    return false;
}
```

