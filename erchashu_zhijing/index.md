# 二叉树的直径


## 二叉树的直径
[二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/description/)
### 方法一：递归
在计算当前二叉树的最大深度时，可以先递归计算出其左子树和右子树的最大深度，然后在 O(1) 时间内计算出当前二叉树的最大深度。递归在访问到空节点时退出。
```golang
func diameterOfBinaryTree(root *TreeNode) (ans int) {
    var dfs func(*TreeNode) int
    dfs = func(node *TreeNode) int {
        if node == nil {
            return -1 // 下面 +1 后，对于叶子节点就刚好是 0
        }
        lLen := dfs(node.Left) + 1  // 左子树最大链长+1
        rLen := dfs(node.Right) + 1 // 右子树最大链长+1
        ans = max(ans, lLen+rLen)   // 两条链拼成路径
        return max(lLen, rLen)      // 当前子树最大链长
    }
    dfs(root)
    return
}

func max(a, b int) int { 
	if a < b { 
		return b 
	}
	return a 
}
```
复杂度分析

时间复杂度：O(n)，其中 n 为二叉树的节点个数。

空间复杂度：O(n)。最坏情况下，二叉树退化成一条链，递归需要 O(n) 的栈空间。

