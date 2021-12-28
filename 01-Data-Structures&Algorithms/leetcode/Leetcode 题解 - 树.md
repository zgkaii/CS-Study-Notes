<!-- MarkdownTOC -->

- [1. 二叉树的前序、中序、后序遍历](#1-二叉树的前序中序后序遍历)
- [2. 二叉树的层序遍历](#2-二叉树的层序遍历)
- [3. 判断树的子结构](#3-判断树的子结构)
- [4. 判断平衡二叉树](#4-判断平衡二叉树)
- [5. 路径总和](#5-路径总和)
- [6. 二叉树的深度](#6-二叉树的深度)
- [7. 二叉树的最近公共祖先](#7-二叉树的最近公共祖先)
- [8. 重建二叉树](#8-重建二叉树)

<!-- /MarkdownTOC --> 

## 1. 二叉树的前序、中序、后序遍历

[ Leetcode94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/) / [Leetcode144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/) / [ Leetcode145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

> 前序遍历： 根节点 -> 左孩子 -> 右孩子
>
> 中序遍历： 左孩子 -> 根节点 -> 右孩子
>
> 后序遍历： 左孩子 -> 右孩子 -> 根节点 

这里以前序遍历为例，讲解其主要几类解法。

- 解法一，递归：时间复杂度O(n) , 空间复杂度O(n)

```java
    public List<Integer> preorderTraversal(TreeNode root) {
		List<Integer> res = new ArrayList<>();
        preorder(root, res);
        return res;
    }

	private void preorder(TreeNode root, List<Integer> res) {
        if (root == null) return;
        
        res.add(root.val);
        res.add(root.left, res);
        res.add(root.right, res);
    }
```

- 解法二，迭代：时间复杂度O(n) , 空间复杂度O(n)。递归隐式地维护了一个栈，在迭代的时候需要显式地将这个栈模拟出来，其余的实现与细节都相同。

```java
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    Deque<TreeNode> stack = new LinkedList<>();
    // 同时满足root为null及stack为空，才将结果返回
    while(root != null || !stack.isEmpty()) {
        // root不为null时入栈，向left方向推进直到叶子节点
        while(root != null) {
            // 首先将当前root的val加入res中，即“根左右”中首先进行的根遍历
            res.add(root.val);
            stack.push(root);
            root = root.left;
        }
        // 向left推进到叶子节点后弹出该叶子节点
        root = stack.pop();
        // 向right推进，如果right也是null，那么回到一开始的while判断栈
        // 是否空，不空则跳过第二个while，再次弹出栈顶。也就是某节点的
        // 左子树遍历完毕，回到该节点。
        root = root.right;
    }
    return res;
}
```



## 2. 二叉树的层序遍历

[Leetcode102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

解法一，广度优先遍历，时间复杂度O(n)，空间复杂度O(n)

```java
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) return res;
        
        // 队列
        LinkedList<TreeNode> queue = new LinkedList<>();
        // 将根节点放入队列中，然后不断遍历队列
        queue.add(root);
        
        while (queue.size() > 0) {
            // 当前一层节点个数
            int size = queue.size();
            // subList存储每层的结点值
            ArrayList<Integer> subList = new ArrayList<>();
            
            for (int i = 0; i < size; ++i) {
                // 出队
                TreeNode node = queue.poll();
                subList.add(node.val);
                // 左右子节点如果不为空就加入到队列中
                if (node.left != null) 
                    queue.add(node.left);
                if (node.right != null) 
                    queue.add(node.right);
            }
            res.add(subList);
        }
        return res;
    }
```



## 3. 判断树的子结构

[剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

```java
    public boolean isSubStructure(TreeNode A, TreeNode B) {
      if (A == null || B == null) return false;
      return check(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B);
    }

    public boolean check(TreeNode A, TreeNode B) {
      // 当节点 B 为空：说明树 B 已匹配完成（越过叶子节点），因此返回 true
      // 当节点 A 为空：说明已经越过树 A 叶子节点，即匹配失败，返回 false 
      // 当节点 A 和 B 的值不同：说明匹配失败，返回 false
      if (B == null) return true;
      if (A == null || A.val != B.val) return false;
      return check(A.left, B.left) && check(A.right, B.right);
    }
```



## 4. 判断平衡二叉树

[110. 平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree/)

- 自顶向下递归，时间O(n^2)，空间O(n)

```java
    public boolean isBalanced(TreeNode root) {
        if (root == null) return true;
        return Math.abs(height(root.left) - height(root.right)) <= 1 
            && isBalanced(root.left) && isBalanced(root.right);
    }

    public int height(TreeNode root) {
        if (root == null) return 0;
        return Math.max(height(root.left), height(root.right)) + 1;
    }
```

- 自下向上递归，时间O(n)，空间O(n)

```java
    public boolean isBalanced(TreeNode root) {
		return height(root) >= 0;
    }

    public int height(TreeNode root) {
        if (root == null) return 0;
        
        int left = height(root.left);
        int right = height(root.right);
        if (left == -1 || right == -1 || Math.abs(left - right) > 1) return -1;
        return Math.max(left, right) + 1;
    }
```



## 5. 路径总和

[113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)

- 深度优先搜索

```java
    List<List<Integer>> res = new LinkedList<>();
    Deque<Integer> path = new LinkedList<>();

    public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
      dfs(root, targetSum);
      return res;
    }

    void dfs(TreeNode root, int targetSum) {
      if (root == null) return;
      path.offerLast(root.val);
      targetSum -= root.val;
      if (targetSum == 0 && root.left == null && root.right == null) {
        res.add(new LinkedList<Integer>(path));
      }
      dfs(root.left, targetSum);
      dfs(root.right, targetSum);
      path.pollLast();
    }
```



## 6. 二叉树的深度

[104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

```java
    public int maxDepth(TreeNode root) {
        if(root == null) return 0;

        int leftDepth = maxDepth(root.left);
        int rightDepth = maxDepth(root.right);

        return Math.max(leftDepth, rightDepth) + 1; 
    }
```



## 7. 二叉树的最近公共祖先

[236. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

```java
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
      if (root == null || root == q || root == p) return root;
      TreeNode left = lowestCommonAncestor(root.left, p, q);
      TreeNode right = lowestCommonAncestor(root.right, p, q);
      return left == null ? right : right == null ? left : root;
    }
```



## 8. 重建二叉树

[105. 从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

```java
    private Map<Integer, Integer> idxMap;

    public TreeNode buildTree(int[] preorder, int[] inorder) {
      int n = preorder.length;
      idxMap = new HashMap<>();
      for (int i = 0; i < n; i++) {
        idxMap.put(inorder[i], i);
      }
      return build(preorder, inorder, 0, n - 1, 0, n - 1);
    }

    public TreeNode build(int[] pre, int[] in, int pre_l, int pre_r, int in_l, int in_r) {
      // preorder = [3, 9, 8, 5, 4, 10, 20, 15, 7]
      // inorder = [4, 5, 8, 10, 9, 3, 15, 20, 7]
      if (pre_l > pre_r) return null;
      // 前序遍历第一个节点就是根节点，中序遍历定位根节点
      int pre_root = pre_l;
      int in_root = idxMap.get(pre[pre_l]);
      // 建立根节点 root = 3
      TreeNode root = new TreeNode(pre[pre_root]);
      // 左子树的节点数目 5
      int size_left = in_root - in_l;
      // 先序遍历「左边界+1 = size_left」
      // (9, 8, 5, 4, 10)对应中序遍历「左边界 开始到 根节点定位-1」(4, 5, 8, 10, 9)
      root.left = build(pre, in, pre_l + 1, pre_l + size_left, in_l, in_root - 1);
      // 先序遍历「左边界+1+左子树节点数目 开始到 右边界」
      // (20, 15, 7)对应中序遍历「根节点定位+1 到 右边界」(15, 20, 7)
      root.right = build(pre, in, pre_l + size_left + 1, pre_r, in_root + 1, in_r);
      return root;
    }
```

