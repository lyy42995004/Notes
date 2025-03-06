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

