## 递归

### 基本概念

程序调用自身的编程**技巧**称为**递归（ recursion）**。递归算法是一种直接或者间接调用自身函数或者方法的算法，它实质上是把一个大型复杂的原问题拆分成具有相同性质的子问题来求解。

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

递归应用广泛，除了运用在裴波拉契数列，阶乘，汉诺塔等问题上（其他应用可查看[Recursion](https://introcs.cs.princeton.edu/java/23recursion/)），还用于处理具有天然的递归性的数据结构的问题，例如链表反转、求树的深度等。

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

迭代（这里其实是动态规划的思想）

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

递归实际上利用了系统堆栈实现自身调用，那么我们也可以通过借助堆栈模拟递归的执行过程或者使用堆栈的循环结构算法。因为递归本身就是通过堆栈实现的，我们只要把递归函数调用的局部变量和相应的状态放入到一个栈结构中，在函数调用和返回时做好push和pop操作。保存中间结果模拟递归过程，将其转为非递归形式。

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

## 分治

## 回溯

分支污染

https://leetcode-cn.com/problems/combination-sum/

## 广度优先遍历

## 深度优先遍历

## 贪心算法

## 动态规划

## 总结

## 参考资料

* [什么是递归，通过这篇文章，让你彻底搞懂递归](https://mp.weixin.qq.com/s?__biz=MzU0ODMyNDk0Mw==&mid=2247487910&idx=1&sn=2670aec7139c6b98e83ff66114ac1cf7&chksm=fb418286cc360b90741ed54fecd62fd45571b2caba3e41473a7ea0934f918d4b31537689c664&token=1327182919&lang=zh_CN#rd)
* [漫谈递归与非递归](https://www.cnblogs.com/bakari/p/5349383.html)
* https://leetcode-cn.com/circle/article/yXFal5/