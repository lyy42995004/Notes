# 中等

## [103. 二叉树的锯齿形层序遍历](https://leetcode.cn/problems/binary-tree-zigzag-level-order-traversal/)

> 给你二叉树的根节点 `root` ，返回其节点值的 **锯齿形层序遍历** 。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。
>
>  
>
> **示例 1：**
>
> <img src="https://assets.leetcode.com/uploads/2021/02/19/tree1.jpg" alt="img" style="zoom:50%;" />
>
> ```
> 输入：root = [3,9,20,null,null,15,7]
> 输出：[[3],[20,9],[15,7]]
> ```
>
> **示例 2：**
>
> ```
> 输入：root = [1]
> 输出：[[1]]
> ```
>
> **示例 3：**
>
> ```
> 输入：root = []
> 输出：[]
> ```
>
>  
>
> **提示：**
>
> - 树中节点数目在范围 `[0, 2000]` 内
> - `-100 <= Node.val <= 100`

与这题类似[429. N 叉树的层序遍历](https://leetcode.cn/problems/n-ary-tree-level-order-traversal/)，只需将偶数层数组进行逆序再加入`ans`中。

```cpp
vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
    if (root == nullptr)
        return {};
    int k = 0;
    vector<vector<int>> ans;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int n = q.size();
        vector<int> tmp;
        for (int i = 0; i < n; ++i) {
            TreeNode* node = q.front();
            q.pop();
            tmp.push_back(node->val);
            if (node->left)
                q.push(node->left);
            if (node->right)
                q.push(node->right);
        }
        if (k % 2 == 1)
            reverse(tmp.begin(), tmp.end());
        k++;
        ans.push_back(tmp);
    }
    return ans;
}
```

## [429. N 叉树的层序遍历](https://leetcode.cn/problems/n-ary-tree-level-order-traversal/)

> 给定一个 N 叉树，返回其节点值的*层序遍历*。（即从左到右，逐层遍历）。
>
> 树的序列化输入是用层序遍历，每组子节点都由 null 值分隔（参见示例）。
>
> 
>
> **示例 1：**
>
> <img src="https://assets.leetcode.com/uploads/2018/10/12/narytreeexample.png" alt="img" style="zoom: 33%;" />
>
> ```
> 输入：root = [1,null,3,2,4,null,5,6]
> 输出：[[1],[3,2,4],[5,6]]
> ```
>
> **示例 2：**
>
> <img src="https://assets.leetcode.com/uploads/2019/11/08/sample_4_964.png" alt="img" style="zoom:50%;" />
>
> ```
> 输入：root = [1,null,2,3,4,5,null,null,6,7,null,8,null,9,10,null,null,11,null,12,null,13,null,null,14]
> 输出：[[1],[2,3,4,5],[6,7,8,9,10],[11,12,13],[14]]
> ```
>
> 
>
> **提示：**
>
> - 树的高度不会超过 `1000`
> - 树的节点总数在 `[0, 10^4]` 之间

BFS，每一次遍历同一层的所有节点。

```cpp
vector<vector<int>> levelOrder(Node* root) {
    if (root == nullptr)
        return {};
    vector<vector<int>> ans;
    queue<Node*> q;
    q.push(root);
    while (!q.empty()) {
        vector<int> tmp;
        int n = q.size();
        // 一层一层出队
        for (int i = 0; i < n; ++i) {
            Node* node = q.front();
            tmp.push_back(node->val);
            q.pop();

            for (Node* child : node->children)
                q.push(child);
        }
        ans.push_back(tmp);
    }
    return ans;
}
```

## [515. 在每个树行中找最大值](https://leetcode.cn/problems/find-largest-value-in-each-tree-row/)

> 给定一棵二叉树的根节点 `root` ，请找出该二叉树中每一层的最大值。
>
>  
>
> **示例1：**
>
> <img src="https://assets.leetcode.com/uploads/2020/08/21/largest_e1.jpg" alt="img" style="zoom:50%;" />
>
> ```
> 输入: root = [1,3,2,5,3,null,9]
> 输出: [1,3,9]
> ```
>
> **示例2：**
>
> ```
> 输入: root = [1,2,3]
> 输出: [1,3]
> ```
>
>  
>
> **提示：**
>
> - 二叉树的节点个数的范围是 `[0,10^4]`
> - `-2^31 <= Node.val <= 2^31 - 1`

**BFS**

```cpp
vector<int> largestValues(TreeNode* root) {
    if (root == nullptr)
        return {};
    vector<int> ans;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int n = q.size();
        int x = INT_MIN;
        while (n--) {
            TreeNode* node = q.front();
            x = max(x, node->val);
            q.pop();
            if (node->left)
                q.push(node->left);
            if (node->right)
                q.push(node->right);
        }
        ans.push_back(x);
    }
    return ans;
}
```

## [662. 二叉树最大宽度](https://leetcode.cn/problems/maximum-width-of-binary-tree/)

> 给你一棵二叉树的根节点 `root` ，返回树的 **最大宽度** 。
>
> 树的 **最大宽度** 是所有层中最大的 **宽度** 。
>
> 每一层的 **宽度** 被定义为该层最左和最右的非空节点（即，两个端点）之间的长度。将这个二叉树视作与满二叉树结构相同，两端点间会出现一些延伸到这一层的 `null` 节点，这些 `null` 节点也计入长度。
>
> 题目数据保证答案将会在 **32 位** 带符号整数范围内。
>
>  
>
> **示例 1：**
>
> <img src="https://assets.leetcode.com/uploads/2021/05/03/width1-tree.jpg" alt="img" style="zoom:50%;" />
>
> ```
> 输入：root = [1,3,2,5,3,null,9]
> 输出：4
> 解释：最大宽度出现在树的第 3 层，宽度为 4 (5,3,null,9) 。
> ```
>
> **示例 2：**
>
> <img src="https://assets.leetcode.com/uploads/2022/03/14/maximum-width-of-binary-tree-v3.jpg" alt="img" style="zoom:50%;" />
>
> ```
> 输入：root = [1,3,2,5,null,null,9,6,null,7]
> 输出：7
> 解释：最大宽度出现在树的第 4 层，宽度为 7 (6,null,null,null,null,null,7) 。
> ```
>
> **示例 3：**
>
> <img src="https://assets.leetcode.com/uploads/2021/05/03/width3-tree.jpg" alt="img" style="zoom:50%;" />
>
> ```
> 输入：root = [1,3,2,5]
> 输出：2
> 解释：最大宽度出现在树的第 2 层，宽度为 2 (3,2) 。
> ```
>
>  
>
> **提示：**
>
> - 树中节点的数目范围是 `[1, 3000]`
> - `-100 <= Node.val <= 100`

**队列 + BFS**

通过 pair 类型将节点对应索引也入队，每次一层一层出队，并记录最左节点，和最右节点的索引，用来计算本层的最大宽度。

使用 `unsigned int` 类型来存储节点索引避免编号溢出的问题，当溢出时两者相减也是正确的宽度。

```cpp
int widthOfBinaryTree(TreeNode* root) {
    unsigned int ans = 0;
    queue<pair<TreeNode*, unsigned int>> q;
    q.push({root, 0});
    while (!q.empty()) {
        int n = q.size();
        auto p = q.front();
        unsigned int left = p.second;
        while (n--) {
            p = q.front();
            TreeNode* node = p.first;
            unsigned int x = p.second;
            q.pop();
            if (node->left)
                q.push({node->left, x * 2});
            if (node->right)
                q.push({node->right, x * 2 + 1});
        }
        unsigned int right = p.second;
        ans = max(ans, right - left + 1);
    }
    return ans;
}
```

**数组 +  BFS**

```cpp
int widthOfBinaryTree(TreeNode* root) {
    unsigned int ans = 0;
    // 数组模拟队列
    vector<pair<TreeNode*, unsigned int>> q;
    q.emplace_back(root, 0);
    while (!q.empty()) {
        // 更新最大宽度
        auto& [lnode, l] = q.front();
        auto& [rnode, r] = q.back();
        ans = max(ans, r - l + 1);

        // 下一层进队
        vector<pair<TreeNode*, unsigned int>> tmp;
        for (auto& [node, x] : q) {
            if (node->left)
                tmp.emplace_back(node->left, 2 * x);
            if (node->right)
                tmp.emplace_back(node->right, 2 * x + 1);
        }
        q = tmp;
    }
    return ans;
}
```

> 结构化绑定是 C++17 引入的特性，能把结构体、数组、`std::pair`等对象 “拆分” 成多个独立变量。
>
> ```cpp
> auto [变量1, 变量2, ...] = 对象
> ```
>
> - `auto` 自动推导变量类型。
> - `[变量1, 变量2, ...]` 是自定义的变量名。
> - `对象` 是要拆分的目标。
