**Flood Fill 算法**是一种用于填充连通区域的算法，常用于图像处理、绘图工具中的“油漆桶”工具，以及解决迷宫、岛屿问题等。它的核心思想是从一个起始点开始，向四周扩散，填充所有符合条件的区域。

**算法思想**：

1. **起始点**：选择一个起始点（通常是某个像素或网格单元）。
2. **填充条件**：定义填充的条件（例如，填充与起始点颜色相同或满足某种属性的区域）。
3. **扩散方式**：从起始点开始，向四周（上下左右或更多方向）递归或迭代地填充符合条件的区域。

**实现方式**：

1. **深度优先搜索（DFS）**：
   - 使用递归或栈实现。
   - 从起始点开始，向四周扩散，直到所有符合条件的区域都被填充。
   - 适合小规模数据，但递归深度过大时可能导致栈溢出。
2. **广度优先搜索（BFS）**：
   - 使用队列实现。
   - 从起始点开始，按层次向四周扩散。
   - 适合大规模数据，不会出现栈溢出问题。

# 简单

## [733. 图像渲染](https://leetcode.cn/problems/flood-fill/)

> 有一幅以 `m x n` 的二维整数数组表示的图画 `image` ，其中 `image[i][j]` 表示该图画的像素值大小。你也被给予三个整数 `sr` , `sc` 和 `color` 。你应该从像素 `image[sr][sc]` 开始对图像进行上色 **填充** 。
>
> 为了完成 **上色工作**：
>
> 1. 从初始像素开始，将其颜色改为 `color`。
> 2. 对初始坐标的 **上下左右四个方向上** 相邻且与初始像素的原始颜色同色的像素点执行相同操作。
> 3. 通过检查与初始像素的原始颜色相同的相邻像素并修改其颜色来继续 **重复** 此过程。
> 4. 当 **没有** 其它原始颜色的相邻像素时 **停止** 操作。
>
> 最后返回经过上色渲染 **修改** 后的图像 。
>
>  
>
> **示例 1:**
>
> ![img](https://assets.leetcode.com/uploads/2021/06/01/flood1-grid.jpg)
>
> **输入：**image = [[1,1,1],[1,1,0],[1,0,1]]，sr = 1, sc = 1, color = 2
>
> **输出：**[[2,2,2],[2,2,0],[2,0,1]]
>
> **解释：**在图像的正中间，坐标 `(sr,sc)=(1,1)` （即红色像素）,在路径上所有符合条件的像素点的颜色都被更改成相同的新颜色（即蓝色像素）。
>
> 注意，右下角的像素 **没有** 更改为2，因为它不是在上下左右四个方向上与初始点相连的像素点。
>
>  
>
> **示例 2:**
>
> **输入：**image = [[0,0,0],[0,0,0]], sr = 0, sc = 0, color = 0
>
> **输出：**[[0,0,0],[0,0,0]]
>
> **解释：**初始像素已经用 0 着色，这与目标颜色相同。因此，不会对图像进行任何更改。
>
>  
>
> **提示:**
>
> - `m == image.length`
> - `n == image[i].length`
> - `1 <= m, n <= 50`
> - `0 <= image[i][j], color < 2^16`
> - `0 <= sr < m`
> - `0 <= sc < n`

```cpp
const int dx[4] = {1, -1, 0, 0};
const int dy[4] = {0, 0, 1, -1};

vector<vector<int>> floodFill(vector<vector<int>>& image, int sr, int sc, int color) {
    int old_color = image[sr][sc];
    if (old_color == color) {
        return image;
    }

    int m = image.size(), n = image[0].size();
    queue<pair<int, int>> que;
    que.emplace(sr, sc);
    image[sr][sc] = color;

    while (!que.empty()) {
        auto [x, y] = que.front();
        que.pop();
        for (int i = 0; i < 4; ++i) {
            int nx = x + dx[i], ny = y + dy[i];
            if (nx >= 0 && nx < m && ny >= 0 && ny < n && image[nx][ny] == old_color) {
                image[nx][ny] = color;
                que.emplace(nx, ny);
            }
        }
    }
    return image;
}
```

# 中等

## [130. 被围绕的区域](https://leetcode.cn/problems/surrounded-regions/)

> 给你一个 `m x n` 的矩阵 `board` ，由若干字符 `'X'` 和 `'O'` 组成，**捕获** 所有 **被围绕的区域**：
>
> - **连接：**一个单元格与水平或垂直方向上相邻的单元格连接。
> - **区域：连接所有** `'O'` 的单元格来形成一个区域。
> - **围绕：**如果您可以用 `'X'` 单元格 **连接这个区域**，并且区域中没有任何单元格位于 `board` 边缘，则该区域被 `'X'` 单元格围绕。
>
> 通过 **原地** 将输入矩阵中的所有 `'O'` 替换为 `'X'` 来 **捕获被围绕的区域**。你不需要返回任何值。
>
>  
>
> **示例 1：**
>
> **输入：**board = [["X","X","X","X"],["X","O","O","X"],["X","X","O","X"],["X","O","X","X"]]
>
> **输出：**[["X","X","X","X"],["X","X","X","X"],["X","X","X","X"],["X","O","X","X"]]
>
> **解释：**
>
> ![img](https://pic.leetcode.cn/1718167191-XNjUTG-image.png)
>
> 在上图中，底部的区域没有被捕获，因为它在 board 的边缘并且不能被围绕。
>
> **示例 2：**
>
> **输入：**board = [["X"]]
>
> **输出：**[["X"]]
>
>  
>
> **提示：**
>
> - `m == board.length`
> - `n == board[i].length`
> - `1 <= m, n <= 200`
> - `board[i][j]` 为 `'X'` 或 `'O'`

从矩形四周遍历，如果是字符`'O'`，则将它标记为`'F'`，后续进行BFS遍历所有未被包围的字符`'O'`，最后再将被标记的字符`'F'`还原为字符`'O'`，未被标记的字符`'O'`改为字符`'X'`。

```cpp
const int dx[4] = {0, 0, 1, -1};
const int dy[4] = {1, -1, 0, 0};

void solve(vector<vector<char>>& board) {
    int m = board.size(), n = board[0].size();
    queue<pair<int, int>> que;
    for (int i = 0; i < m; ++i) {
        if (board[i][0] == 'O') {
            que.emplace(i, 0);
            board[i][0] = 'F';
        }
        if (board[i][n - 1] == 'O') {
            que.emplace(i, n - 1);
            board[i][n - 1] = 'F';
        }
    }
    for (int i = 1; i < n - 1; ++i) {
        if (board[0][i] == 'O') {
            que.emplace(0, i);
            board[0][i] = 'F';
        }
        if (board[m - 1][i] == 'O') {
            que.emplace(m - 1, i);
            board[m - 1][i] = 'F';
        }
    }
    while (!que.empty()) {
        auto [x, y] = que.front();
        que.pop();
        for (int k = 0; k < 4; ++k) {
            int nx = x + dx[k], ny = y + dy[k];
            if (nx >= 0 && nx < m && ny >= 0 && ny < n && board[nx][ny] == 'O') {
                que.emplace(nx, ny);
                board[nx][ny] = 'F';
            }
        }
    }
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (board[i][j] == 'F') {
                board[i][j] = 'O';
            } else if (board[i][j] == 'O') {
                board[i][j] = 'X';
            }
        }
    }
}
```

## [200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)

> 给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。
>
> 岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。
>
> 此外，你可以假设该网格的四条边均被水包围。
>
>  
>
> **示例 1：**
>
> ```
> 输入：grid = [
>   ["1","1","1","1","0"],
>   ["1","1","0","1","0"],
>   ["1","1","0","0","0"],
>   ["0","0","0","0","0"]
> ]
> 输出：1
> ```
>
> **示例 2：**
>
> ```
> 输入：grid = [
>   ["1","1","0","0","0"],
>   ["1","1","0","0","0"],
>   ["0","0","1","0","0"],
>   ["0","0","0","1","1"]
> ]
> 输出：3
> ```
>
>  
>
> **提示：**
>
> - `m == grid.length`
> - `n == grid[i].length`
> - `1 <= m, n <= 300`
> - `grid[i][j]` 的值为 `'0'` 或 `'1'`

`for`循环遍历数组，每遇到一个字符`'1'`，通过BFS遍历上下左右所有的字符`'1'`，标记为已遍历，并且`ans++`。

通过将访问过的陆地标记为 `'0'`，可以避免额外的 `visited` 数组。

```cpp
const int dx[4] = {1, -1, 0, 0};
const int dy[4] = {0, 0, 1, -1};

int numIslands(vector<vector<char>>& grid) {
    int m = grid.size(), n = grid[0].size();
    queue<pair<int, int>> que;
    int ans = 0;
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (grid[i][j] == '1') {
                ans++;
                que.emplace(i, j);
                grid[i][j] = '0';
                while (!que.empty()) {
                    auto [x, y] = que.front();
                    que.pop();
                    for (int k = 0; k < 4; ++k) {
                        int nx = x + dx[k], ny = y + dy[k];
                        if (nx >= 0 && nx < m && ny >= 0 && ny < n && grid[nx][ny] == '1') {
                            grid[nx][ny] = '0';
                            que.emplace(nx, ny);
                        }
                    }
                }
            }
        }
    }
    return ans;
}
```

## [695. 岛屿的最大面积](https://leetcode.cn/problems/max-area-of-island/)

> 给你一个大小为 `m x n` 的二进制矩阵 `grid` 。
>
> **岛屿** 是由一些相邻的 `1` (代表土地) 构成的组合，这里的「相邻」要求两个 `1` 必须在 **水平或者竖直的四个方向上** 相邻。你可以假设 `grid` 的四个边缘都被 `0`（代表水）包围着。
>
> 岛屿的面积是岛上值为 `1` 的单元格的数目。
>
> 计算并返回 `grid` 中最大的岛屿面积。如果没有岛屿，则返回面积为 `0` 。
>
>  
>
> **示例 1：**
>
> ![img](https://assets.leetcode.com/uploads/2021/05/01/maxarea1-grid.jpg)
>
> ```
> 输入：grid = [[0,0,1,0,0,0,0,1,0,0,0,0,0],[0,0,0,0,0,0,0,1,1,1,0,0,0],[0,1,1,0,1,0,0,0,0,0,0,0,0],[0,1,0,0,1,1,0,0,1,0,1,0,0],[0,1,0,0,1,1,0,0,1,1,1,0,0],[0,0,0,0,0,0,0,0,0,0,1,0,0],[0,0,0,0,0,0,0,1,1,1,0,0,0],[0,0,0,0,0,0,0,1,1,0,0,0,0]]
> 输出：6
> 解释：答案不应该是 11 ，因为岛屿只能包含水平或垂直这四个方向上的 1 。
> ```
>
> **示例 2：**
>
> ```
> 输入：grid = [[0,0,0,0,0,0,0,0]]
> 输出：0
> ```
>
>  
>
> **提示：**
>
> - `m == grid.length`
> - `n == grid[i].length`
> - `1 <= m, n <= 50`
> - `grid[i][j]` 为 `0` 或 `1`

与这题类似[200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)，只不过每次统计一下岛屿的大小并保留最大值。

```cpp
const int dx[4] = {0, 0, 1, -1};
const int dy[4] = {1, -1, 0, 0};

int maxAreaOfIsland(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    queue<pair<int, int>> que;
    int ans = 0;
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (grid[i][j] == 1) {
                que.emplace(i, j);
                grid[i][j] = 0;
                int cnt = 0;
                while (!que.empty()) {
                    cnt++;
                    auto [x, y] = que.front();
                    que.pop();
                    for (int k = 0; k < 4; ++k) {
                        int nx = x + dx[k], ny = y + dy[k];
                        if (nx >= 0 && nx < m && ny >= 0 && ny < n && grid[nx][ny] == 1) {
                            que.emplace(nx, ny);
                            grid[nx][ny] = 0;
                        }
                    }
                }
                ans = max(ans, cnt);
            }
        }
    }
    return ans;
}
```

## [994. 腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/)

> 在给定的 `m x n` 网格 `grid` 中，每个单元格可以有以下三个值之一：
>
> - 值 `0` 代表空单元格；
> - 值 `1` 代表新鲜橘子；
> - 值 `2` 代表腐烂的橘子。
>
> 每分钟，腐烂的橘子 **周围 4 个方向上相邻** 的新鲜橘子都会腐烂。
>
> 返回 *直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`* 。
>
>  
>
> **示例 1：**
>
> **![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/02/16/oranges.png)**
>
> ```
> 输入：grid = [[2,1,1],[1,1,0],[0,1,1]]
> 输出：4
> ```
>
> **示例 2：**
>
> ```
> 输入：grid = [[2,1,1],[0,1,1],[1,0,1]]
> 输出：-1
> 解释：左下角的橘子（第 2 行， 第 0 列）永远不会腐烂，因为腐烂只会发生在 4 个方向上。
> ```
>
> **示例 3：**
>
> ```
> 输入：grid = [[0,2]]
> 输出：0
> 解释：因为 0 分钟时已经没有新鲜橘子了，所以答案就是 0 。
> ```
>
>  
>
> **提示：**
>
> - `m == grid.length`
> - `n == grid[i].length`
> - `1 <= m, n <= 10`
> - `grid[i][j]` 仅为 `0`、`1` 或 `2`

```cpp
const int dx[4] = {1, -1, 0, 0};
const int dy[4] = {0, 0, 1, -1};  

int orangesRotting(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    int cnt = 0; // 新鲜橘子个数
    queue<pair<int, int>> que;
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (grid[i][j] == 2) {
                que.emplace(i, j);
            } else if (grid[i][j] == 1) {
                cnt++;
            }
        }
    }
    if (cnt == 0) {
        return 0;
    }

    int ans = 0;
    while (!que.empty()) {
        ans++;
        int size = que.size();
        for (int i = 0; i < size; ++i) {
            auto [x, y] = que.front(); 
            que.pop();
            for (int k = 0; k < 4; ++k) {
                int nx = x + dx[k], ny = y + dy[k];
                if (nx >= 0 && nx < m && ny >= 0 && ny < n && grid[nx][ny] == 1) {
                    grid[nx][ny] = 2;
                    que.emplace(nx, ny);
                    cnt--;
                }
            }
        }
    }
    return cnt == 0 ? ans-1 : -1;
}
```

## [1020. 飞地的数量](https://leetcode.cn/problems/number-of-enclaves/)

> 给你一个大小为 `m x n` 的二进制矩阵 `grid` ，其中 `0` 表示一个海洋单元格、`1` 表示一个陆地单元格。
>
> 一次 **移动** 是指从一个陆地单元格走到另一个相邻（**上、下、左、右**）的陆地单元格或跨过 `grid` 的边界。
>
> 返回网格中 **无法** 在任意次数的移动中离开网格边界的陆地单元格的数量。
>
>  
>
> **示例 1：**
>
> <img src="https://assets.leetcode.com/uploads/2021/02/18/enclaves1.jpg" alt="img" style="zoom:50%;" />
>
> ```
> 输入：grid = [[0,0,0,0],[1,0,1,0],[0,1,1,0],[0,0,0,0]]
> 输出：3
> 解释：有三个 1 被 0 包围。一个 1 没有被包围，因为它在边界上。
> ```
>
> **示例 2：**
>
> <img src="https://assets.leetcode.com/uploads/2021/02/18/enclaves2.jpg" alt="img" style="zoom:50%;" />
>
> ```
> 输入：grid = [[0,1,1,0],[0,0,1,0],[0,0,1,0],[0,0,0,0]]
> 输出：0
> 解释：所有 1 都在边界上或可以到达边界。
> ```
>
>  
>
> **提示：**
>
> - `m == grid.length`
> - `n == grid[i].length`
> - `1 <= m, n <= 500`
> - `grid[i][j]` 的值为 `0` 或 `1`

与这题[130. 被围绕的区域](https://leetcode.cn/problems/surrounded-regions/)类型。

```cpp
const int dx[4] = {1, -1, 0, 0};
const int dy[4] = {0, 0, 1, -1};

int numEnclaves(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    queue<pair<int, int>> que;
    for (int i = 0; i < m; ++i) {
        if (grid[i][0] == 1) {
            que.emplace(i, 0);
            grid[i][0] = 0;
        }
        if (grid[i][n - 1] == 1) {
            que.emplace(i, n - 1);
            grid[i][n - 1] = 0;
        }
    }
    for (int i = 1; i < n - 1; ++i) {
        if (grid[0][i] == 1) {
            que.emplace(0, i);
            grid[0][i] = 0;
        }
        if (grid[m - 1][i] == 1) {
            que.emplace(m - 1, i);
            grid[m - 1][i] = 0;
        }
    }
    while (!que.empty()) {
        auto [x, y] = que.front();
        que.pop();
        for (int k = 0; k < 4; ++k) {
            int nx = x + dx[k], ny = y + dy[k];
            if (nx >= 0 && nx < m && ny >= 0 && ny < n && grid[nx][ny] == 1) {
                que.emplace(nx, ny);
                grid[nx][ny] = 0;
            }
        }
    }
    int ans = 0;
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (grid[i][j] == 1) {
                ans++;
            }
        }
    }
    return ans; 
}
```

# [208. 实现 Trie (前缀树)](https://leetcode.cn/problems/implement-trie-prefix-tree/)

> **[Trie](https://baike.baidu.com/item/字典树/9825209?fr=aladdin)**（发音类似 "try"）或者说 **前缀树** 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补全和拼写检查。
>
> 请你实现 Trie 类：
>
> - `Trie()` 初始化前缀树对象。
> - `void insert(String word)` 向前缀树中插入字符串 `word` 。
> - `boolean search(String word)` 如果字符串 `word` 在前缀树中，返回 `true`（即，在检索之前已经插入）；否则，返回 `false` 。
> - `boolean startsWith(String prefix)` 如果之前已经插入的字符串 `word` 的前缀之一为 `prefix` ，返回 `true` ；否则，返回 `false` 。
>
>  
>
> **示例：**
>
> ```
> 输入
> ["Trie", "insert", "search", "search", "startsWith", "insert", "search"]
> [[], ["apple"], ["apple"], ["app"], ["app"], ["app"], ["app"]]
> 输出
> [null, null, true, false, true, null, true]
> 
> 解释
> Trie trie = new Trie();
> trie.insert("apple");
> trie.search("apple");   // 返回 True
> trie.search("app");     // 返回 False
> trie.startsWith("app"); // 返回 True
> trie.insert("app");
> trie.search("app");     // 返回 True
> ```
>
>  
>
> **提示：**
>
> - `1 <= word.length, prefix.length <= 2000`
> - `word` 和 `prefix` 仅由小写英文字母组成
> - `insert`、`search` 和 `startsWith` 调用次数 **总计** 不超过 `3 * 10^4` 次

```cpp
class Trie {
public:
    Trie() : children(26), isEnd(false) {}
    
    void insert(string word) {
        Trie* node = this;
        for (char ch : word) {
            ch -= 'a';
            if (node->children[ch] == nullptr) {
                node->children[ch] = new Trie();
            }
            node = node->children[ch];
        }
        node->isEnd = true;
    }
    
    bool search(string word) {
        Trie* node = searchPrefix(word);
        return node != nullptr && node->isEnd;
    }
    
    bool startsWith(string prefix) {
        return searchPrefix(prefix) != nullptr;
    }
private:
    vector<Trie*> children;
    bool isEnd;

    Trie* searchPrefix(string prefix) {
        Trie* node = this;
        for (char ch : prefix) {
            ch -= 'a';
            if (node->children[ch] == nullptr) {
                return nullptr;
            }
            node = node->children[ch];
        }
        return node;
    }
};
```

