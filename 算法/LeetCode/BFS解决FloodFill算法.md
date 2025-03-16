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
                            grid[nx][ny] = '0'; // 立即修改，否则可能会导致重复入队
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

