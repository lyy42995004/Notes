**拓扑排序**

拓扑排序(Topological Sorting)是针对有向无环图(DAG)的一种线性排序算法，它将图中的所有顶点排成一个线性序列，使得对于图中的每一条有向边 (u, v)，u 在序列中总是位于 v 的前面。

**特性**：

- 只适用于有向无环图(DAG)
- 一个DAG可能有多个拓扑排序结果
- 如果图中存在环，则无法进行拓扑排序

**BFS 解决拓扑排序方法**

1. **计算所有节点的入度**：统计每个节点的入边数量。
2. **初始化队列**：将所有入度为0的节点加入队列。
3. **BFS处理**：
   - 从队列中取出一个节点，加入结果列表。
   - 将该节点的所有邻居节点的入度减1。
   - 如果某个邻居节点的入度变为0，将其加入队列。
4. **检查结果**：如果结果列表包含所有节点，则成功；否则说明图中存在环。

# 中等

## [207. 课程表](https://leetcode.cn/problems/course-schedule/)

> 你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。
>
> 在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程 `bi` 。
>
> - 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0` ，你需要先完成课程 `1` 。
>
> 请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。
>
>  
>
> **示例 1：**
>
> ```
> 输入：numCourses = 2, prerequisites = [[1,0]]
> 输出：true
> 解释：总共有 2 门课程。学习课程 1 之前，你需要完成课程 0 。这是可能的。
> ```
>
> **示例 2：**
>
> ```
> 输入：numCourses = 2, prerequisites = [[1,0],[0,1]]
> 输出：false
> 解释：总共有 2 门课程。学习课程 1 之前，你需要先完成课程 0 ；并且学习课程 0 之前，你还应先完成课程 1 。这是不可能的。
> ```
>
>  
>
> **提示：**
>
> - `1 <= numCourses <= 2000`
> - `0 <= prerequisites.length <= 5000`
> - `prerequisites[i].length == 2`
> - `0 <= ai, bi < numCourses`
> - `prerequisites[i]` 中的所有课程对 **互不相同**

```cpp
bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
    vector<vector<int>> edges(numCourses); // 邻接表
    vector<int> in(numCourses); // 每个点的入度

    // 建表
    for (auto& v : prerequisites) {
        int a = v[0], b = v[1];
        edges[a].push_back(b);
        in[b]++;
    }
    // 入度为0的节点入队
    queue<int> que;
    for (int i = 0; i < in.size(); ++i) {
        if (in[i] ==  0) {
            que.push(i);
        }
    }
    // bfs
    while (!que.empty()) {
        int i = que.front(); que.pop();
        for (int x : edges[i]) {
            in[x]--;
            if (in[x] == 0) {
                que.push(x);
            }
        }
    }

    for (int i : in) {
        if (i != 0) {
            return false;
        }
    }
    return true;
}
```

## [210. 课程表 II](https://leetcode.cn/problems/course-schedule-ii/)

> 现在你总共有 `numCourses` 门课需要选，记为 `0` 到 `numCourses - 1`。给你一个数组 `prerequisites` ，其中 `prerequisites[i] = [ai, bi]` ，表示在选修课程 `ai` 前 **必须** 先选修 `bi` 。
>
> - 例如，想要学习课程 `0` ，你需要先完成课程 `1` ，我们用一个匹配来表示：`[0,1]` 。
>
> 返回你为了学完所有课程所安排的学习顺序。可能会有多个正确的顺序，你只要返回 **任意一种** 就可以了。如果不可能完成所有课程，返回 **一个空数组** 。
>
>  
>
> **示例 1：**
>
> ```
> 输入：numCourses = 2, prerequisites = [[1,0]]
> 输出：[0,1]
> 解释：总共有 2 门课程。要学习课程 1，你需要先完成课程 0。因此，正确的课程顺序为 [0,1] 。
> ```
>
> **示例 2：**
>
> ```
> 输入：numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
> 输出：[0,2,1,3]
> 解释：总共有 4 门课程。要学习课程 3，你应该先完成课程 1 和课程 2。并且课程 1 和课程 2 都应该排在课程 0 之后。
> 因此，一个正确的课程顺序是 [0,1,2,3] 。另一个正确的排序是 [0,2,1,3] 。
> ```
>
> **示例 3：**
>
> ```
> 输入：numCourses = 1, prerequisites = []
> 输出：[0]
> ```
>
>  
>
> **提示：**
>
> - `1 <= numCourses <= 2000`
> - `0 <= prerequisites.length <= numCourses * (numCourses - 1)`
> - `prerequisites[i].length == 2`
> - `0 <= ai, bi < numCourses`
> - `ai != bi`
> - 所有`[ai, bi]` **互不相同**

```cpp
vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
    vector<vector<int>> edges(numCourses);
    vector<int> in(numCourses);

    for (auto& v : prerequisites) {
        int a = v[0], b = v[1];
        edges[b].push_back(a);
        in[a]++;
    }

    queue<int> que;
    for (int i = 0; i < numCourses; ++i) {
        if(in[i] == 0) {
            que.push(i);
        }
    }

    vector<int> ans;
    while (!que.empty()) {
        int i = que.front(); que.pop();
        ans.push_back(i);
        for (int x : edges[i]) {
            in[x]--;
            if (in[x] == 0) {
                que.push(x);
            }
        }
    }
    for (int i = 0; i < numCourses; ++i) {
        if (in[i] != 0) {
            return {};
        }
    }
    return ans;
}
```

## [LCR 114. 火星词典](https://leetcode.cn/problems/Jf1JuT/)

> 现有一种使用英语字母的外星文语言，这门语言的字母顺序与英语顺序不同。
>
> 给定一个字符串列表 `words` ，作为这门语言的词典，`words` 中的字符串已经 **按这门新语言的字母顺序进行了排序** 。
>
> 请你根据该词典还原出此语言中已知的字母顺序，并 **按字母递增顺序** 排列。若不存在合法字母顺序，返回 `""` 。若存在多种可能的合法字母顺序，返回其中 **任意一种** 顺序即可。
>
> 字符串 `s` **字典顺序小于** 字符串 `t` 有两种情况：
>
> - 在第一个不同字母处，如果 `s` 中的字母在这门外星语言的字母顺序中位于 `t` 中字母之前，那么 `s` 的字典顺序小于 `t` 。
> - 如果前面 `min(s.length, t.length)` 字母都相同，那么 `s.length < t.length` 时，`s` 的字典顺序也小于 `t` 。
>
>  
>
> **示例 1：**
>
> ```
> 输入：words = ["wrt","wrf","er","ett","rftt"]
> 输出："wertf"
> ```
>
> **示例 2：**
>
> ```
> 输入：words = ["z","x"]
> 输出："zx"
> ```
>
> **示例 3：**
>
> ```
> 输入：words = ["z","x","z"]
> 输出：""
> 解释：不存在合法字母顺序，因此返回 ""。
> ```
>
>  
>
> **提示：**
>
> - `1 <= words.length <= 100`
> - `1 <= words[i].length <= 100`
> - `words[i]` 仅由小写英文字母组成
>
>  
>
> 注意：本题与主站 269 题相同： https://leetcode-cn.com/problems/alien-dictionary/

```cpp
unordered_map<char, unordered_set<char>> edges;
unordered_map<char, int> in;

string alienOrder(vector<string>& words) {
    for (auto& s : words) {
        for (auto ch : s) {
            in[ch] = 0;
        }
    }
    int n = words.size();
    for (int i = 0; i < n; ++i) {
        for (int j = i + 1; j < n; ++j) {
            if (add(words[i], words[j]) == false) {
                return "";
            }
        }
    }

    queue<char> que;
    for (auto& [a, b] : in) {
        if (b == 0) {
            que.push(a);
        }
    }
    string ans;
    while (!que.empty()) {
        char t = que.front(); que.pop();
        ans += t;
        for (char ch : edges[t]) {
            --in[ch];
            if (in[ch] == 0) {
                que.push(ch);
            }
        }
    }
    for (auto& [a, b] : in) {
        if (b != 0) {
            return "";
        }
    }
    return ans;
}

bool add(const string& s1, const string& s2) {
    int len1 = s1.size(), len2 = s2.size();
    int n = min(len1, len2);
    int i = 0;
    while (i < n) {
        if (s1[i] != s2[i]) {
            char ch1 = s1[i], ch2 = s2[i];
            if (!edges.count(ch1) || !edges[ch1].count(ch2)) {
                edges[ch1].insert(ch2);
                ++in[ch2];
            }
            break;
        }
        i++;
    }
    // 如果 s1 是 s2 的前缀且 s1 比 s2 长（如 "abc" 在 "ab" 之后），则非法
    if (i == n && len1 > len2) {
        return false;
    }
    return true;
}
```

