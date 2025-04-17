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
> - `0 <= image[i][j], color < 216`
> - `0 <= sr < m`
> - `0 <= sc < n`

```cpp
const int dx[4] = {0, 0, -1, 1};
const int dy[4] = {1, -1, 0, 0};

vector<vector<int>> floodFill(vector<vector<int>>& image, int sr, int sc, int color) {
    if (image[sr][sc] != color) {
        int old_color = image[sr][sc];
        image[sr][sc] = color;
        dfs(image, sr, sc, old_color, color);        
    }
    return image;
}

void dfs(vector<vector<int>>& image, int x, int y, int old_color, int color) {
    int n = image.size(), m = image[0].size();
    for (int k = 0; k < 4; ++k) {
        int nx = x + dx[k], ny = y + dy[k];
        if (nx >= 0 && nx < n && ny >= 0 && ny < m 
            && image[nx][ny] == old_color) {
            image[nx][ny] = color;
            dfs(image, nx, ny, old_color, color);
        }
    }
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
> <img src="https://pic.leetcode.cn/1718167191-XNjUTG-image.png" alt="img" style="zoom:67%;" />
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

```cpp
const int dx[4] = {0, 0, 1, -1};
const int dy[4] = {1, -1, 0, 0};

void solve(vector<vector<char>>& board) {
    int m = board.size(), n = board[0].size();
    queue<pair<int, int>> que;
    for (int i = 0; i < m; ++i) {
        if (board[i][0] == 'O') {
            dfs(board, i, 0);
        }
        if (board[i][n - 1] == 'O') {
            dfs(board, i, n - 1);
        }
    }
    for (int i = 1; i < n - 1; ++i) {
        if (board[0][i] == 'O') {
            dfs(board, 0, i);
        }
        if (board[m - 1][i] == 'O') {
            dfs(board, m - 1, i);
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

void dfs(vector<vector<char>>& board, int x, int y) {
    int m = board.size(), n = board[0].size();
    board[x][y] = 'F';
    for (int k = 0; k < 4; ++k) {
        int nx = x + dx[k], ny = y + dy[k];
        if (nx >= 0 && nx < m && ny >= 0 && ny < n 
            && board[nx][ny] == 'O') {
            dfs(board, nx, ny);
        }
    }
}
```

## [200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)

>给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。
>
>岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。
>
>此外，你可以假设该网格的四条边均被水包围。
>
> 
>
>**示例 1：**
>
>```
>输入：grid = [
>  ["1","1","1","1","0"],
>  ["1","1","0","1","0"],
>  ["1","1","0","0","0"],
>  ["0","0","0","0","0"]
>]
>输出：1
>```
>
>**示例 2：**
>
>```
>输入：grid = [
>  ["1","1","0","0","0"],
>  ["1","1","0","0","0"],
>  ["0","0","1","0","0"],
>  ["0","0","0","1","1"]
>]
>输出：3
>```
>
> 
>
>**提示：**
>
>- `m == grid.length`
>- `n == grid[i].length`
>- `1 <= m, n <= 300`
>- `grid[i][j]` 的值为 `'0'` 或 `'1'`

```cpp
const int dx[4] = {1, -1, 0, 0};
const int dy[4] = {0, 0, 1, -1};

int numIslands(vector<vector<char>>& grid) {
    int ans = 0;
    int n = grid.size(), m = grid[0].size();
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < m; ++j) {
            if (grid[i][j] == '1') {
                grid[i][j] = '0';
                dfs(grid, i, j);
                ans++;
            }
        }
    }
    return ans;
}


void dfs (vector<vector<char>>& grid, int x, int y) {
    int n = grid.size(), m = grid[0].size();
    for (int k = 0; k < 4; ++k) {
        int nx = x + dx[k], ny = y + dy[k];
        if (nx >= 0 && nx < n && ny >= 0 && ny < m 
            && grid[nx][ny] == '1') {
            grid[nx][ny] = '0';
            dfs(grid, nx, ny);
        }
    }
}
```

## [417. 太平洋大西洋水流问题](https://leetcode.cn/problems/pacific-atlantic-water-flow/)

> 有一个 `m × n` 的矩形岛屿，与 **太平洋** 和 **大西洋** 相邻。 **“太平洋”** 处于大陆的左边界和上边界，而 **“大西洋”** 处于大陆的右边界和下边界。
>
> 这个岛被分割成一个由若干方形单元格组成的网格。给定一个 `m x n` 的整数矩阵 `heights` ， `heights[r][c]` 表示坐标 `(r, c)` 上单元格 **高于海平面的高度** 。
>
> 岛上雨水较多，如果相邻单元格的高度 **小于或等于** 当前单元格的高度，雨水可以直接向北、南、东、西流向相邻单元格。水可以从海洋附近的任何单元格流入海洋。
>
> 返回网格坐标 `result` 的 **2D 列表** ，其中 `result[i] = [ri, ci]` 表示雨水从单元格 `(ri, ci)` 流动 **既可流向太平洋也可流向大西洋** 。
>
>  
>
> **示例 1：**
>
> ![img](https://assets.leetcode.com/uploads/2021/06/08/waterflow-grid.jpg)
>
> ```
> 输入: heights = [[1,2,2,3,5],[3,2,3,4,4],[2,4,5,3,1],[6,7,1,4,5],[5,1,1,2,4]]
> 输出: [[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]
> ```
>
> **示例 2：**
>
> ```
> 输入: heights = [[2,1],[1,2]]
> 输出: [[0,0],[0,1],[1,0],[1,1]]
> ```
>
>  
>
> **提示：**
>
> - `m == heights.length`
> - `n == heights[r].length`
> - `1 <= m, n <= 200`
> - `0 <= heights[r][c] <= 10^5`

```cpp
const int dx[4] = {0, 0, 1, -1};
const int dy[4] = {1, -1, 0, 0};

vector<vector<int>> pacificAtlantic(vector<vector<int>>& heights) {
    int m = heights.size(), n = heights[0].size();
    vector<vector<bool>> pac(m, vector<bool>(n));
    vector<vector<bool>> atl(m, vector<bool>(n));
    for (int i = 0; i < m; ++i) {
        dfs(heights, i, 0, pac);
        dfs(heights, i, n - 1, atl);
    }
    for (int j = 0; j < n; ++j) {
        dfs(heights, 0, j, pac);
        dfs(heights, m - 1, j, atl);
    }
    vector<vector<int>> ans;
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (pac[i][j] && atl[i][j]) {
                ans.push_back({i, j});
            }
        }
    }
    return ans;
}

void dfs(vector<vector<int>>& heights, int x, int y, vector<vector<bool>>& vis) {
    vis[x][y] = true;
    int m = heights.size(), n = heights[0].size();
    for (int k = 0; k < 4; ++k) {
        int nx = x + dx[k], ny = y + dy[k];
        if (nx >= 0 && nx < m && ny >= 0 && ny < n 
            && heights[nx][ny] >= heights[x][y] && vis[nx][ny] == false) {
            dfs(heights, nx, ny, vis);
        }
    }
}
```

## [529. 扫雷游戏](https://leetcode.cn/problems/minesweeper/)

> 让我们一起来玩扫雷游戏！
>
> 给你一个大小为 `m x n` 二维字符矩阵 `board` ，表示扫雷游戏的盘面，其中：
>
> - `'M'` 代表一个 **未挖出的** 地雷，
> - `'E'` 代表一个 **未挖出的** 空方块，
> - `'B'` 代表没有相邻（上，下，左，右，和所有4个对角线）地雷的 **已挖出的** 空白方块，
> - **数字**（`'1'` 到 `'8'`）表示有多少地雷与这块 **已挖出的** 方块相邻，
> - `'X'` 则表示一个 **已挖出的** 地雷。
>
> 给你一个整数数组 `click` ，其中 `click = [clickr, clickc]` 表示在所有 **未挖出的** 方块（`'M'` 或者 `'E'`）中的下一个点击位置（`clickr` 是行下标，`clickc` 是列下标）。
>
> 根据以下规则，返回相应位置被点击后对应的盘面：
>
> 1. 如果一个地雷（`'M'`）被挖出，游戏就结束了- 把它改为 `'X'` 。
> 2. 如果一个 **没有相邻地雷** 的空方块（`'E'`）被挖出，修改它为（`'B'`），并且所有和其相邻的 **未挖出** 方块都应该被递归地揭露。
> 3. 如果一个 **至少与一个地雷相邻** 的空方块（`'E'`）被挖出，修改它为数字（`'1'` 到 `'8'` ），表示相邻地雷的数量。
> 4. 如果在此次点击中，若无更多方块可被揭露，则返回盘面。
>
>  
>
> **示例 1：**
>
> ![img](https://assets.leetcode.com/uploads/2023/08/09/untitled.jpeg)
>
> ```
> 输入：board = [["E","E","E","E","E"],["E","E","M","E","E"],["E","E","E","E","E"],["E","E","E","E","E"]], click = [3,0]
> 输出：[["B","1","E","1","B"],["B","1","M","1","B"],["B","1","1","1","B"],["B","B","B","B","B"]]
> ```
>
> **示例 2：**
>
> ![img](https://assets.leetcode.com/uploads/2023/08/09/untitled-2.jpeg)
>
> ```
> 输入：board = [["B","1","E","1","B"],["B","1","M","1","B"],["B","1","1","1","B"],["B","B","B","B","B"]], click = [1,2]
> 输出：[["B","1","E","1","B"],["B","1","X","1","B"],["B","1","1","1","B"],["B","B","B","B","B"]]
> ```
>
>  
>
> **提示：**
>
> - `m == board.length`
> - `n == board[i].length`
> - `1 <= m, n <= 50`
> - `board[i][j]` 为 `'M'`、`'E'`、`'B'` 或数字 `'1'` 到 `'8'` 中的一个
> - `click.length == 2`
> - `0 <= clickr < m`
> - `0 <= clickc < n`
> - `board[clickr][clickc]` 为 `'M'` 或 `'E'`

```cpp
const int dy[8] = {0, 0, 1, 1, 1, -1, -1, -1};
const int dx[8] = {1, -1, 0, 1, -1, 0, 1, -1};

vector<vector<char>> updateBoard(vector<vector<char>>& board,
                                 vector<int>& click) {
    int m = board.size(), n = board[0].size();
    int x = click[0], y = click[1];
    int num = count(board, x, y);
    if (board[x][y] == 'M') {
        board[x][y] = 'X';
    } else if (num != 0) {
        board[x][y] = num + '0';
    } else {
        dfs(board, x, y);
    }
    return board;
}

void dfs(vector<vector<char>>& board, int x, int y) {
    int m = board.size(), n = board[0].size();
    board[x][y] = 'B';
    for (int k = 0; k < 8; ++k) {
        int nx = x + dx[k], ny = y + dy[k];
        if (nx >= 0 && nx < m && ny >= 0 && ny < n) {
            int num = count(board, nx, ny);
            if (num == 0 && board[nx][ny] != 'B') {
                dfs(board, nx, ny);
            } else if (num != 0 && board[nx][ny] != 'M') {
                board[nx][ny] = num + '0';
            }
        }
    }
}

int count(vector<vector<char>>& board, int x, int y) {
    int m = board.size(), n = board[0].size();
    int sum = 0;
    for (int k = 0; k < 8; ++k) {
        int nx = x + dx[k], ny = y + dy[k];
        if (nx >= 0 && nx < m && ny >= 0 && ny < n && board[nx][ny] == 'M') {
            ++sum;
        }
    }
    return sum;
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

```cpp
const int dx[4] = {0, 0, 1, -1};
const int dy[4] = {1, -1, 0, 0};
int cnt = 0;

int maxAreaOfIsland(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    int ans = 0;
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (grid[i][j] == 1) {
                cnt = 1;
                grid[i][j] = 0;
                dfs(grid, i, j);
                ans = max(cnt, ans);
            }
        }
    }
    return ans;
}

void dfs(vector<vector<int>>& grid, int x, int y) {
    int m = grid.size(), n = grid[0].size();
    for (int k = 0; k < 4; ++k) {
        int nx = x + dx[k], ny = y + dy[k];
        if (nx >= 0 && nx < m && ny >= 0 && ny < n 
            && grid[nx][ny] == 1) {
            grid[nx][ny] = 0;
            cnt++;
            dfs(grid, nx, ny);
        }
    }
}
```

## [LCR 130. 衣橱整理](https://leetcode.cn/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

> 家居整理师将待整理衣橱划分为 `m x n` 的二维矩阵 `grid`，其中 `grid[i][j]` 代表一个需要整理的格子。整理师自 `grid[0][0]` 开始 **逐行逐列** 地整理每个格子。
>
> 整理规则为：在整理过程中，可以选择 **向右移动一格** 或 **向下移动一格**，但不能移动到衣柜之外。同时，不需要整理 `digit(i) + digit(j) > cnt` 的格子，其中 `digit(x)` 表示数字 `x` 的各数位之和。
>
> 请返回整理师 **总共需要整理多少个格子**。
>
>  
>
> **示例 1：**
>
> ```
> 输入：m = 4, n = 7, cnt = 5
> 输出：18
> ```
>
>  
>
> **提示：**
>
> - `1 <= n, m <= 100`
> - `0 <= cnt <= 20`

```cpp
int ans;
vector<vector<bool>> vis;

const int dx[2] = {0, 1};
const int dy[2] = {1, 0};

int wardrobeFinishing(int m, int n, int cnt) {
    vis = vector<vector<bool>>(m, vector<bool>(n));
    dfs(0, 0, m, n, cnt);
    return ans;
}

void dfs(int x, int y, int m, int n, int cnt) {
    vis[x][y] = true;
    ans++;
    for (int k = 0; k < 2; ++k) {
        int nx = x + dx[k], ny = y + dy[k];
        if (nx >= 0 && nx < m && ny >= 0 && ny < n &&
            vis[nx][ny] == false && digit(nx) + digit(ny) <= cnt) {
            dfs(nx, ny, m, n, cnt);
        }
    }
}

int digit(int x) {
    int ret = 0;
    while (x) {
        ret += x % 10;
        x /= 10;
    }
    return ret;
}
```

