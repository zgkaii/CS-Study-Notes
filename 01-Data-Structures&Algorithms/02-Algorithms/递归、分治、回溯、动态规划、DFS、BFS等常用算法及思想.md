## 递归

### 基本概念

程序调用自身的**编程技巧**称为**递归（ recursion）**。递归算法是一种直接或者间接调用自身函数或者方法的算法，它实质上是把一个大型复杂的原问题拆分成具有相同性质的子问题来求解。

递归至少满足以下两个条件：

- 在过程或函数内调用自身；
- 必须有一个明确的递归终止条件。

以阶乘为例，n的阶乘`n! = n*(n-1)*... *1 (n>0)`，用Java代码表示：

```java
    public static int factorial(int n) { 
        if (n == 1) 
            return 1; 
        return n * factorial(n-1); 
    }
```

那么计算`5!`时程序的执行过程（单向递归？）如下：

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
    public static int fibonacci(int n) {
        if (n <= 1) 
            return n;
        return fibonacci(n-1) + fibonacci(n-2);
    } 
```

以计算`fibonacci(5)` 为例，一层一层的分解过程画成图，递归树如下：

<img src="..\..\images\algorithms\递归树.png" alt="img" style="zoom: 67%;" />

### 应用场景

递归应用广泛，除了运用在裴波拉契数列，阶乘，汉诺塔等问题上（可查看[Recursion](https://introcs.cs.princeton.edu/java/23recursion/)），还用于处理具有天然的递归性的数据结构的问题，例如链表反转、求树的深度等。

解决递归问题时，主要就是确定**递归终止条件 与 （当前层）逻辑处理 和 （下一层）递归调用**，大致模板如下：

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

> 这里是分治还是纯递归？？

## 迭代

**迭代（iteration）**是重复反馈过程的活动，其目的通常是为了接近并到达所需的目标或结果。 每一次对过程的重复被称为一次“迭代”，而每一次迭代得到的结果会被用来作为下一次迭代的初始值。

利用迭代算法解决问题，需要做好以下三个方面的工作：

* 确定迭代变量
* 建立迭代关系式
* 对迭代过程进行控制

还是以计算n的阶乘`n!`为例，先计算1乘2，然后得到结果再乘以3，在用得到结果乘以4......一直乘到n。用Java代码表示：

```java
    public static int factorial(int n) { 
        int res = 1;						// 1.迭代变量
        for (int i = 2; i < n; i++) {		// 3.迭代过程进行控制
            res *= i;					    // 2.建立迭代关系式
        }
        return res; 
    }
```

### 递归与迭代区别

递归是一个**树结构**，可以将其理解为重复“递推”和“回归”的过程，当“递推”到达底部时就会开始“回归”；而迭代是一个**环结构**，从初始迭代变量开始，每次迭代都更新迭代变量，多次迭代直到到达结束状态。

理论上递归和迭代时间复杂度方面是一样的，但实际应用中（函数调用和函数调用堆栈的开销）递归比迭代效率要低。

<img src="..\..\images\algorithms\递归与迭代区别.png" alt="img" style="zoom:80%;" />

### 递归与迭代的转换

理论上递归和迭代可以相互转换，从上面的n的阶乘的问题也可以看出。但实际从算法结构来说，递归声明的结构并不总能转换为迭代结构（原因有待研究）。迭代可以转换为递归，但递归不一定能转换为迭代。

将递归算法转换为非递归算法有两种方法，一种是直接求值（迭代），不需要回溯；另一种是不能直接求值，需要回溯。前者使用一些变量保存中间结果，称为直接转换法，后者使用栈保存中间结果，称为间接转换法。

**直接转换法**

直接转换法通常用来消除**尾递归**（tail recursion）和**单向递归**，不借助堆栈结构将递归转化为迭代结构来替代。（单向递归 → 尾递归 → 迭代）

尾递归函数递归调用返回时正好是函数的结尾，因此递归调用时就不需要保留当前栈帧，可以直接将当前栈帧覆盖掉。

<img src="..\..\images\algorithms\尾递归.png" alt="img" style="zoom: 80%;" />

还是以斐波拉契数列为例。

尾递归：统计步数从简单情境递增到n。

```java
int fib(int n, int i, int pre, int cur)
{
    if (n <= 2)
        return 1;
    else if (i == n)
        return cur;
    else return fib(n, i + 1, cur, pre + cur);
}

fib(5, 2, 1, 1)
fib(5, 3, 1, 2)
fib(5, 4, 2 ,3)
fib(5, 5, 3, 5)
```

尾递归：统计步数从n递减到简单情境。

```java
int fib1(int i, int pre, int cur)
{
    if (i <= 2)
        return cur;
    else return fib1(i - 1, cur, pre + cur);
}

fib1(5, 1, 1)
fib1(4, 1, 2)
fib1(3, 2, 3)
fib1(2, 3, 5)
```

迭代（下面其实使用了动态规划的思想）

```java
    public int factorial(int n) {
        if (n <= 1) {
            return n;
        }
        int n1 = 0, n2 = 0, res = 1;
        for (int i = 2; i <= n; ++i) {
            n1 = n2; 
            n2 = res; 
            res = n1 + n2;
        }
        return res;
    }
```

**间接转换法**

递归实际上利用了系统堆栈实现自身调用，那么我们也可以通**过借助堆栈模拟递归的执行过程或者使用堆栈的循环结构算法**。因为递归本身就是通过堆栈实现的，我们只要把递归函数调用的局部变量和相应的状态放入到一个栈结构中，在函数调用和返回时做好push和pop操作。保存中间结果模拟递归过程，将其转为非递归形式。

```java
    public int climbStairs(int n) {
        if(n < 3) return n;
        Stack<Integer> st = new Stack<Integer>();
        
        st.push(new Integer(1));
        st.push(new Integer(2));
        
        int i = 3;
        while(i <= n){
            Integer a = st.peek();
            st.pop();
            Integer b = st.peek();
            st.push(a);
            st.push(new Integer(a.intValue() + b.intValue()));
            i++;
        }
        return st.peek().intValue();
    }
```

> 编写中...

## 深度优先搜索与广度优先搜索

### 深度优先搜索

**深度优先搜索（Depth-First-Search，简称DFS）**与是一种用于遍历或搜索树或图的算法。

### 广度优先搜索

**广度优先搜索算法（Breadth-First Search，简称为BFS）**，又译作**宽度优先搜索**，或**横向优先搜索**，也是一种图形搜索算法。

### 经典举例

## 分治

### 基本概念

**分治算法（divide and conquer）**和核心思想正如其字面含义，**分而治之**，就是把一个复杂问题分成**两个或者更多**的相同和相似的问题，直到最后问题可以简单的直接求解，原问题的解即是子问题解的合并。

这个定义看起来类似递归的定义，区别在于**分治算法是一种处理问题的思想，而递归是一种编程技巧**。实际上，**分治算法一般比较适合用递归来实现，当然也可以用迭代来实现**，这也是区别之一。

分治算法能解决的问题一般满足下面几个条件：

* 分解：将原问题分解成若干个子问题。
* 分解终止条件：分解的子问题可以独立求解，子问题之间没有相关性。
* 合并：将子问题的解合并，形成原问题的解。（这个合并操作的复杂度不易过高）

### 经典举例

## 回溯

### 基本概念

回溯法采用试错的思想，它尝试分步去解决一个问题。在分步解决问题的过程中，它通过尝试发现现有的分布答案不能得到有效的正确的解答的时候，它将取消上一步甚至上几步的计算，再通过其他的的可能的分布解答再次尝试寻找问题的答案。

回溯法通常使用最简单的递归方法实现。在反反复复上述的步骤后可能出现两种情况：

* 找到一个可能存在的正确答案；
* 在尝试了所有可能的分步方法后宣告该问题没有答案。

在最坏的情况下，回溯法会导致一次复杂度为指数时间的计算。

### 经典举例

分支污染

https://leetcode-cn.com/problems/combination-sum/

## 贪心算法

### 基本概念

贪心算法（Greedy）是在每一步选择中都采取在当前状态下最好或者最优（即最有利的）选择，从而希望导致结果是全局最好或最优的算法。

贪心算法与动态规划的不同在于它对每个子问题的解决方案都做出选择，不能回退。动态规划则会保存以前运算结果，并根据以前的结果对当前进行选择，有回退功能。

适用贪心算法的场景：

简单地说，问题能够分解成子问题解决，子问题的最优解能递推到最终问题的最终解。这种子问题最优解称为最优子结构。

### 经典举例

## 动态规划

### 基本概念

### 经典举例

## 总结

《算法之道》对其中的三种算法进行了归纳总结，如下表所示：
|                  | **标准分治**       | 动态规划           | 贪心算法           |
| ---------------- | ------------------ | ------------------ | ------------------ |
| 适用类型         | 通用问题           | 优化问题           | 优化问题           |
| 子问题结构       | 每个子问题不同     | 很多子问题重复     | 只有一个子问题     |
| 最优子结构       | 不需要             | 必须满足           | 必须满足           |
| 子问题数         | 全部子问题都要解决 | 全部子问题都要解决 | 只要解决一个子问题 |
| 子问题在最优解里 | 全部               | 部分               | 部分               |
| 选择与求解次序   | 先选择后解决子问题 | 先解决子问题后选择 | 先选择后解决子问题 |

## 参考资料

* [什么是递归，通过这篇文章，让你彻底搞懂递归](https://mp.weixin.qq.com/s?__biz=MzU0ODMyNDk0Mw==&mid=2247487910&idx=1&sn=2670aec7139c6b98e83ff66114ac1cf7&chksm=fb418286cc360b90741ed54fecd62fd45571b2caba3e41473a7ea0934f918d4b31537689c664&token=1327182919&lang=zh_CN#rd)
* [漫谈递归与非递归](https://www.cnblogs.com/bakari/p/5349383.html)
* https://leetcode-cn.com/circle/article/yXFal5/