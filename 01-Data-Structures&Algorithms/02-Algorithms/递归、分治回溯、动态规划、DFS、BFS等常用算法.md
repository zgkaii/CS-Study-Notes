## 递归

### 基本概念

程序调用自身的编程**技巧**称为**递归（ recursion）**。递归算法是一种直接或者间接调用自身函数或者方法的算法，它实质上是把一个大型复杂的原问题拆分成具有相同性质的子问题来求解。

以阶乘为例：n的阶乘`n! = n*(n-1)*... *1 (n>0)`，用Java代码表示：

```java
    public static long factorial(int n) { 
        if (n == 1) 
            return 1; 
        return n * factorial(n-1); 
    }
```

那么计算5！时程序的执行过程如下：

```html
factorial(5) 
   factorial(4) 
      factorial(3) 
         factorial(2) 
            factorial(1) 
               return 1 
            return 2*1 = 2 
         return 3*2 = 6 
      return 4*6 = 24 
   return 5*24 = 120
```

又如裴波拉契（Fibonacci）数列：`0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, ...,`其递推公式为：`F(n) = F(n−1) + F(n−2) 其中(n > 2), F(1) = 1, F(0) = 0`,用Java代码表示：

```java
    public static long fibonacci(int n) {
        if (n <= 1) return n;
        return fibonacci(n-1) + fibonacci(n-2);
    } 
```

以计算`fibonacci(5)` 为例，一层一层的分解过程画成图，递归树如下：

<img src="..\..\images\algorithms\递归树.png" alt="img" style="zoom: 67%;" />

### 应用场景

递归应用广泛，除了运用在裴波拉契数列，阶乘，汉诺塔等问题上（其他应用可查看[Recursion](https://introcs.cs.princeton.edu/java/23recursion/)），还用于处理具有天然的递归性的数据结构的问题，例如链表反转、求树的深度等。

解决递归问题时，主要就是确定**递归终止条件 与 当前逻辑处理 和 （下一层）递归调用**，大致模板如下：

```java
public void recursion(参数0) {
    if (终止条件) {    // 必须
        return;
    }
    
    逻辑处理 		  // 非必须
    recursion(参数1) // 递归调用（必须）
    ...
    逻辑处理 		  // 非必须
}
```

以树的前序遍历（[PreOrder Traversal in a binary tree](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)）为例：

```java
    public List<Integer> preorderTraversal(TreeNode1 root) {
        List<Integer> list = new ArrayList<>();
        preorder(root, list);
        return list;
    }

    private void preorder(TreeNode1 root, List<Integer> list) {
        // 终止条件
        if (root == null) {
            return;
        }
        // 逻辑处理
        list.add(root.val);
        // 递归调用
        preorder(root.left, list);
        preorder(root.right, list);
    }
```

## 迭代

## 分治

## 回溯

分治污染

https://leetcode-cn.com/problems/combination-sum/

## 广度优先遍历

## 深度优先遍历

## 贪心算法

## 动态规划

## 总结

## 参考资料

> https://leetcode-cn.com/circle/article/yXFal5/
>
> https://ayesawyer.github.io/2019/05/06/%E5%B8%B8%E7%94%A8%E7%AE%97%E6%B3%95%E4%B9%8B%E9%80%92%E5%BD%92%E3%80%81%E5%88%86%E6%B2%BB%E3%80%81%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E7%AD%89/

* [什么是递归，通过这篇文章，让你彻底搞懂递归](https://mp.weixin.qq.com/s?__biz=MzU0ODMyNDk0Mw==&mid=2247487910&idx=1&sn=2670aec7139c6b98e83ff66114ac1cf7&chksm=fb418286cc360b90741ed54fecd62fd45571b2caba3e41473a7ea0934f918d4b31537689c664&token=1327182919&lang=zh_CN#rd)