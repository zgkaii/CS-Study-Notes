<!-- GFM-TOC -->
* [1. 给表达式加括号](#1-给表达式加括号)
* [2. 不同的二叉搜索树](#2-不同的二叉搜索树)
<!-- GFM-TOC -->


# 1. 给表达式加括号

241\. Different Ways to Add Parentheses (Medium)

[Leetcode](https://leetcode.com/problems/different-ways-to-add-parentheses/description/) / [力扣](https://leetcode-cn.com/problems/different-ways-to-add-parentheses/description/)

```html
Input: "2-1-1".

((2-1)-1) = 0
(2-(1-1)) = 2

Output : [0, 2]
```

```java
    Map<String, List<Integer>> map = new HashMap<>();

    public List<Integer> diffWaysToCompute(String input) {
        if (map.containsKey(input)) return map.get(input);
        List<Integer> res = new LinkedList<>();
        for (int i = 0; i < input.length(); i++) {
            char ch = input.charAt(i);
            if (ch == '-' || ch == '*' || ch == '+') {
                // 分解子问题 （1）2*3-4与5（2）2*3与4*5（3）2与3-4*5
                String part1 = input.substring(0, i);
                String part2 = input.substring(i + 1);
                // 进一步分解子问题，直到不能分解 （1.1）2*3与4 （1.2）2与3-4 ...
                List<Integer> leftPart = diffWaysToCompute(input.substring(0, i));
                List<Integer> rightPart = diffWaysToCompute(input.substring(0, i));
                // 组合计算
                for (Integer l : leftPart) {
                    for (Integer r : rightPart) {
                        if(ch == '+') 
                            res.add(l + r);
                        if(ch == '-')
                            res.add(l - r);
                        if(ch == '*')
                            res.add(l * r);    
                    }
                }
            }
        }
        if (res.isEmpty()) res.add(Integer.valueOf(input));
        map.put(input, res); // 添加映射以减少进行重复计算的时间
        return res;
    }
```

# 2. 不同的二叉搜索树

95\. Unique Binary Search Trees II (Medium)

[Leetcode](https://leetcode.com/problems/unique-binary-search-trees-ii/description/) / [力扣](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/description/)

给定一个数字 n，要求生成所有值为 1...n 的二叉搜索树。

```html
Input: 3
Output:
[
  [1,null,3,2],
  [3,2,null,1],
  [3,1,null,null,2],
  [2,1,3],
  [1,null,2,null,3]
]
Explanation:
The above output corresponds to the 5 unique BST's shown below:

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

```java
public List<TreeNode> generateTrees(int n) {
    if (n < 1) {
        return new LinkedList<TreeNode>();
    }
    return generateSubtrees(1, n);
}

private List<TreeNode> generateSubtrees(int s, int e) {
    List<TreeNode> res = new LinkedList<TreeNode>();
    if (s > e) {
        res.add(null);
        return res;
    }
    for (int i = s; i <= e; ++i) {
        List<TreeNode> leftSubtrees = generateSubtrees(s, i - 1);
        List<TreeNode> rightSubtrees = generateSubtrees(i + 1, e);
        for (TreeNode left : leftSubtrees) {
            for (TreeNode right : rightSubtrees) {
                TreeNode root = new TreeNode(i);
                root.left = left;
                root.right = right;
                res.add(root);
            }
        }
    }
    return res;
}
```
