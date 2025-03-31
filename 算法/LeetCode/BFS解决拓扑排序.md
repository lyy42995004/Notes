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

