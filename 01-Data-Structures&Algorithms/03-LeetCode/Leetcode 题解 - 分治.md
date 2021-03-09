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

# 3.数组中的逆序对

[[剑指 Offer 51. 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。

```java
    public int reversePairs(int[] nums) {
        if (nums.length < 2) 
            return 0;
        int[] tmp = new int[nums.length];// 临时数组用于归并
        return mergeSort(nums, tmp, 0, nums.length - 1);
    }

    public int mergeSort(int[] nums, int[] tmp, int left, int right) {
        // 终止条件,子数组长度为1，停止划分
        if (left >= right) 
            return 0;
        // 递归划分左子数组与右子数组
        int mid = left + right >>> 1;
        int res = mergeSort(nums, tmp, left, mid) + mergeSort(nums, tmp, mid + 1, right);
        //  合并阶段 idx为临时数组的移动指针
        int i = left, j = mid + 1, idx = left, count = 0;
        // 左右两数组都还剩有数字未排序时
        while (i <= mid && j <= right) {
            if (nums[i] > nums[j]) {
                tmp[idx++] = nums[j];
                count = j - mid; // 统计j->mid 之间，比nums[i]小的元素个数
                j++;
            } else {
                tmp[idx++] = nums[i];
                res += count;
                i++;
            }
        }
        // 左右两数组有一边已经移动完毕，剩下另一边可进行快速移动
        while (i <= mid) {
            tmp[idx++] = nums[i];
            res += count;
            i++;
        }
        
        while (j <= right) 
            tmp[idx++] = nums[j++];

        // 将tep中的数还原回原nums中
        for (int k = left; k <= right; ++k)
            nums[k] = tmp[k];
        
        return res;
    }
```

更好理解的写法：

```java
    public int reversePairs(int[] nums) {
        if (nums == null || nums.length < 2)
            return 0;
        int[] tmp = new int[nums.length];// 临时数组用于归并
        return mergeSort(nums, tmp, 0, nums.length - 1);
    }
 
    public int mergeSort(int[] nums, int[] tmp, int left, int right) {
        if (left == right) // 终止条件,子数组长度为1，停止划分
            return 0;
        int mid = left + (right - left >>> 1);
        // 递归划分左子数组与右子数组
        return mergeSort(nums, tmp, left, mid) 
            + mergeSort(nums, tmp, mid + 1, right) 
            + mergeCross(nums, tmp, left, mid, right);
    }
	// 合并阶段
    public int mergeCross(int[] nums, int[] tmp, int left, int mid, int right) {
        // pos待归并数组的起始位置
        int pos = 0, i = left, j = mid + 1, res = 0;
		// 左右两个子区间都未遍历完
        while (i <= mid && j <= right) {
            res += nums[i] <= nums[j] ? 0 : mid - i + 1;
            tmp[pos++] = nums[i] <= nums[j] ? nums[i++] : nums[j++];
        }
        // 左半部分元素有剩余
        while (i <= mid)
            tmp[pos++] = nums[i++];
		// 右半部分元素有剩余
        while (j <= right) 
            tmp[pos++] = nums[j++];
        // 用临时数组tmp赋值回numsk
        for (int k = left; k <= right; k++) {
            nums[k] = tmp[i - left];
        }
        return res;
    }
```

> 时间复杂度：O(nlog n)，空间复杂度：O(n)