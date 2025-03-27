广度优先搜索(BFS, Breadth-First Search)是一种用于图和树结构的遍历算法，特别适合解决无权图的**最短路径问题**。

**算法思想**：

BFS从起始节点开始，按照"广度优先"的原则，逐层向外扩展搜索：

1. 先访问起始节点的所有相邻节点(第一层)。
2. 然后访问这些相邻节点的相邻节点(第二层)。
3. 依此类推，直到找到目标节点或遍历完整个图。

**BFS最短路径算法步骤**：

1. **初始化**：
   - 创建队列，放入起始节点。
   - 记录每个节点的访问状态和距离(起始节点距离为0)。
2. **循环处理队列**：
   - 取出队首节点。
   - 遍历其所有未访问的邻居节点。
   - 记录这些邻居的距离(当前节点距离+1)。
   - 将邻居节点加入队列。
3. **终止条件**：
   - 找到目标节点时返回其距离。
   - 队列为空时表示无法到达。

# 中等

## [433. 最小基因变化](https://leetcode.cn/problems/minimum-genetic-mutation/)

> 基因序列可以表示为一条由 8 个字符组成的字符串，其中每个字符都是 `'A'`、`'C'`、`'G'` 和 `'T'` 之一。
>
> 假设我们需要调查从基因序列 `start` 变为 `end` 所发生的基因变化。一次基因变化就意味着这个基因序列中的一个字符发生了变化。
>
> - 例如，`"AACCGGTT" --> "AACCGGTA"` 就是一次基因变化。
>
> 另有一个基因库 `bank` 记录了所有有效的基因变化，只有基因库中的基因才是有效的基因序列。（变化后的基因必须位于基因库 `bank` 中）
>
> 给你两个基因序列 `start` 和 `end` ，以及一个基因库 `bank` ，请你找出并返回能够使 `start` 变化为 `end` 所需的最少变化次数。如果无法完成此基因变化，返回 `-1` 。
>
> 注意：起始基因序列 `start` 默认是有效的，但是它并不一定会出现在基因库中。
>
>  
>
> **示例 1：**
>
> ```
> 输入：start = "AACCGGTT", end = "AACCGGTA", bank = ["AACCGGTA"]
> 输出：1
> ```
>
> **示例 2：**
>
> ```
> 输入：start = "AACCGGTT", end = "AAACGGTA", bank = ["AACCGGTA","AACCGCTA","AAACGGTA"]
> 输出：2
> ```
>
> **示例 3：**
>
> ```
> 输入：start = "AAAAACCC", end = "AACCCCCC", bank = ["AAAACCCC","AAACCCCC","AACCCCCC"]
> 输出：3
> ```
>
>  
>
> **提示：**
>
> - `start.length == 8`
> - `end.length == 8`
> - `0 <= bank.length <= 10`
> - `bank[i].length == 8`
> - `start`、`end` 和 `bank[i]` 仅由字符 `['A', 'C', 'G', 'T']` 组成

```cpp
int minMutation(string startGene, string endGene, vector<string>& bank) {
    if (startGene == endGene) {
        return 0;
    }

    unordered_set<string> vis;
    unordered_set<string> hash(bank.begin(), bank.end());
    const string change = "ACGT";

    queue<string> que;
    que.push(startGene);
    hash.insert(startGene);

    int step = 0;
    while (!que.empty()) {
        step++;
        int n = que.size();
        while (n--) {
            string str = que.front();
            que.pop();

            for (int i = 0; i < 8; ++i) {
                string tmp = str;
                for (int j = 0; j < 4; ++j) {
                    tmp[i] = change[j];
                    if (hash.count(tmp) && !vis.count(tmp)) {
                        if (tmp == endGene) {
                            return step;
                        }
                        vis.insert(tmp);
                        que.push(tmp);
                    }
                }
            }
        }
    }
    return -1;
}
```

## [1926. 迷宫中离入口最近的出口](https://leetcode.cn/problems/nearest-exit-from-entrance-in-maze/)

> 给你一个 `m x n` 的迷宫矩阵 `maze` （**下标从 0 开始**），矩阵中有空格子（用 `'.'` 表示）和墙（用 `'+'` 表示）。同时给你迷宫的入口 `entrance` ，用 `entrance = [entrancerow, entrancecol]` 表示你一开始所在格子的行和列。
>
> 每一步操作，你可以往 **上**，**下**，**左** 或者 **右** 移动一个格子。你不能进入墙所在的格子，你也不能离开迷宫。你的目标是找到离 `entrance` **最近** 的出口。**出口** 的含义是 `maze` **边界** 上的 **空格子**。`entrance` 格子 **不算** 出口。
>
> 请你返回从 `entrance` 到最近出口的最短路径的 **步数** ，如果不存在这样的路径，请你返回 `-1` 。
>
>  
>
> **示例 1：**
>
> <img src="https://assets.leetcode.com/uploads/2021/06/04/nearest1-grid.jpg" alt="img" style="zoom:67%;" />
>
> ```
> 输入：maze = [["+","+",".","+"],[".",".",".","+"],["+","+","+","."]], entrance = [1,2]
> 输出：1
> 解释：总共有 3 个出口，分别位于 (1,0)，(0,2) 和 (2,3) 。
> 一开始，你在入口格子 (1,2) 处。
> - 你可以往左移动 2 步到达 (1,0) 。
> - 你可以往上移动 1 步到达 (0,2) 。
> 从入口处没法到达 (2,3) 。
> 所以，最近的出口是 (0,2) ，距离为 1 步。
> ```
>
> **示例 2：**
>
> <img src="https://assets.leetcode.com/uploads/2021/06/04/nearesr2-grid.jpg" alt="img" style="zoom:67%;" />
>
> ```
> 输入：maze = [["+","+","+"],[".",".","."],["+","+","+"]], entrance = [1,0]
> 输出：2
> 解释：迷宫中只有 1 个出口，在 (1,2) 处。
> (1,0) 不算出口，因为它是入口格子。
> 初始时，你在入口与格子 (1,0) 处。
> - 你可以往右移动 2 步到达 (1,2) 处。
> 所以，最近的出口为 (1,2) ，距离为 2 步。
> ```
>
> **示例 3：**
>
> <img src="https://assets.leetcode.com/uploads/2021/06/04/nearest3-grid.jpg" alt="img" style="zoom:67%;" />
>
> ```
> 输入：maze = [[".","+"]], entrance = [0,0]
> 输出：-1
> 解释：这个迷宫中没有出口。
> ```
>
>  
>
> **提示：**
>
> - `maze.length == m`
> - `maze[i].length == n`
> - `1 <= m, n <= 100`
> - `maze[i][j]` 要么是 `'.'` ，要么是 `'+'` 。
> - `entrance.length == 2`
> - `0 <= entrancerow < m`
> - `0 <= entrancecol < n`
> - `entrance` 一定是空格子。

BFS层序遍历，每次`step++`，第一次到达边界时就是最短路径，`vis`数组判断是否访问过该节点。

```cpp
const int dx[4] = {0, 0, 1, -1};
const int dy[4] = {1, -1, 0, 0};

int nearestExit(vector<vector<char>>& maze, vector<int>& entrance) {
    int m = maze.size(), n = maze[0].size();
    vector<vector<bool>> vis(m, vector<bool>(n, false));

    queue<pair<int, int>> que;
    que.emplace(entrance[0], entrance[1]);
    vis[entrance[0]][entrance[1]] = true;

    int step = 0;
    while (!que.empty()) {
        step++;
        int cnt = que.size();
        while (cnt--) {
            auto [x, y] = que.front();
            que.pop();
            for (int k = 0; k < 4; ++k) {
                int nx = x + dx[k], ny = y + dy[k];
                if (nx >= 0 && nx < m && ny >= 0 && ny < n && !vis[nx][ny]) {
                    vis[nx][ny] = true;
                    if (maze[nx][ny] == '.') {
                        if (nx == 0 || nx == m - 1 || ny == 0 || ny == n - 1) {
                            return step;
                        }
                        que.emplace(nx, ny);
                    }
                }
            }
        }
    }
    return -1;
}
```

# 困难

## [127. 单词接龙](https://leetcode.cn/problems/word-ladder/)