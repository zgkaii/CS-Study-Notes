<!-- MarkdownTOC -->
- [1. 用栈实现队列](#1-用栈实现队列)
- [2. 用队列实现栈](#2-用队列实现栈)
- [3. 最小值栈](#3-最小值栈)
- [4. 有效的括号](#4-有效的括号)
- [5. 每日温度](#5-每日温度)
- [6. 下一个更大元素 II](#6-下一个更大元素-ii)
- [7. 柱状图中最大的矩形](#7-柱状图中最大的矩形)
- [8. 接雨水](#8-接雨水)

<!-- /MarkdownTOC -->
# 1. 用栈实现队列

232\. Implement Queue using Stacks (Easy)

[Leetcode](https://leetcode.com/problems/implement-queue-using-stacks/description/) / [力扣](https://leetcode-cn.com/problems/implement-queue-using-stacks/description/)

栈的顺序为后进先出，而队列的顺序为先进先出。使用两个栈实现队列，一个元素需要经过两个栈才能出队列，在经过第一个栈时元素顺序被反转，经过第二个栈时再次被反转，此时就是先进先出顺序。

```java
public class MyQueue {
    private Stack<Integer> in = new Stack<>();
    private Stack<Integer> out = new Stack<>();
    private int front;

    public MyQueue() {
        in = new Stack<>();
        out = new Stack<>();
    }

    public void push(int x) {
        if (in.empty()) front = x;
        in.push(x);
    }

    public int pop() {
        if (out.isEmpty()) {
            while (!in.isEmpty())
                out.push(in.pop());
        }
        return out.pop();
    }

    public int peek() {
        if (out.isEmpty()) return front;
        return out.peek();
    }

    public boolean empty() {
        return in.isEmpty() && out.isEmpty();
    }
}
```

# 2. 用队列实现栈

225\. Implement Stack using Queues (Easy)

[Leetcode](https://leetcode.com/problems/implement-stack-using-queues/description/) / [力扣](https://leetcode-cn.com/problems/implement-stack-using-queues/description/)

在将一个元素 x 插入队列时，为了维护原来的后进先出顺序，需要让 x 插入队列首部。而队列的默认插入顺序是队列尾部，因此在将 x 插入队列尾部之后，需要让除了 x 之外的所有元素出队列，再入队列。

```java
class MyStack {
    private Queue<Integer> queue;

    public MyStack() {
        queue = new LinkedList<>();
    }

    public void push(int x) {
        queue.add(x);
        int cnt = queue.size();
        while (cnt-- > 1) {
            queue.add(queue.poll());
        }
    }

    public int pop() {
        return queue.remove();
    }

    public int top() {
        return queue.peek();
    }

    public boolean empty() {
        return queue.isEmpty();
    }
}
```

# 3. 最小值栈

155\. Min Stack (Easy)

[Leetcode](https://leetcode.com/problems/min-stack/description/) / [力扣](https://leetcode-cn.com/problems/min-stack/description/)

```java
public class MinStack {
    int min = Integer.MAX_VALUE;
    Stack<Integer> stack = new Stack<>();

    public void push(int x) {
        if (x <= min) {
            stack.push(min);
            min = x;
        }
        stack.push(x);
    }

    public void pop() {
        if (stack.pop() == min) {
            min = stack.pop();
        }
    }

    public int top() {
        return stack.peek();
    }

    public int getMin() {
        return min;
    }
}
```

# 4. 有效的括号

20\. Valid Parentheses (Easy)

[Leetcode](https://leetcode.com/problems/valid-parentheses/description/) / [力扣](https://leetcode-cn.com/problems/valid-parentheses/description/)

```html
"()[]{}"

Output : true
```

```java
        Stack<Character> stack = new Stack<Character>();
        for (char c : s.toCharArray()) {
            if (c == '(') {
                stack.push(')');
            } else if (c == '{') {
                stack.push('}');
            } else if (c == '[') {
                stack.push(']');
            } else if (stack.isEmpty() || stack.pop() != c) {
                return false;
            }
        }
        return stack.isEmpty();
```

# 5. 每日温度

739\. Daily Temperatures (Medium)

[Leetcode](https://leetcode.com/problems/daily-temperatures/description/) / [力扣](https://leetcode-cn.com/problems/daily-temperatures/description/)

```html
Input: [73, 74, 75, 71, 69, 72, 76, 73]
Output: [1, 1, 4, 2, 1, 1, 0, 0]
```

单调栈：

* 时间复杂度O(n)，空间复杂度O(n)

在遍历数组时用栈把数组中的数存起来，如果当前遍历的数比栈顶元素来的大，说明栈顶元素的下一个比它大的数就是当前元素。

```java
    public int[] dailyTemperatures(int[] T) {
        if (T == null || T.length == 0) return null;

        int[] res = new int[T.length];
        Stack<Integer> stack = new Stack<>();
        for (int cur = 0; cur < T.length; cur++) {
            while (!stack.isEmpty() && T[cur] > T[stack.peek()]) {
                int pre = stack.pop();
                res[pre] = cur - pre;
            }
            stack.add(cur);
        }
        return res;
    }
```

# 6. 下一个更大元素 II

503\. Next Greater Element II (Medium)

[Leetcode](https://leetcode.com/problems/next-greater-element-ii/description/) / [力扣](https://leetcode-cn.com/problems/next-greater-element-ii/description/)

与 739. Daily Temperatures (Medium) 不同的是，数组是循环数组，并且最后要求的不是距离而是下一个元素。

```java
    public int[] nextGreaterElements(int[] nums) {
        int N = nums.length;
        int[] res = new int[N];
        Arrays.fill(res, -1);
        Stack<Integer> stack = new Stack<>();
        for (int i = 0; i < N * 2; i++) {
            int num = nums[i % N];
            while (!stack.isEmpty() && nums[stack.peek()] < num) {
                res[stack.pop()] = num;
            }
            if (i < N) stack.push(i % N);
        }
        return res;
    }
```

# 7. 柱状图中最大的矩形

84\. Largest Rectangle in Histogram (Hard)

[Leetcode](https://leetcode.com/problems/largest-rectangle-in-histogram/) / [力扣](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

单调栈：

* 时间复杂度O(n)，空间复杂度O(n)

```java
    public int largestRectangleArea(int[] h) {
		// 维护一个单调栈，栈内存储数组下标，只有和h[left] > right = s.peek()时,才能确定left元素的右边界
        // [2,1,5,6,2,3]
        // left = 0  =>  stack:0  
        // left = 1  =>  stack:null  height:2 width:1 => Area:1  stack:1
		// left = 2  =>  										 stack:1,2
        // left = 3  =>  										 stack:1,2,3
        // left = 4  =>  stack:1,2   height:6 width:1 => Area:6  stack:1,2,4
        // left = 5  =>  									     stack:1,2,4,5	  
        // 							 height:3 width:1 => Area:3  stack:1,2,4
        // 						     height:2 width:3 => Area:6  stack:1,2
        // 							 height:5 width:2 => Area:10 stack:1
        // 							 height:1 width:6 => Area:6  stack:null       
        int len = h.length, left = 0, maxArea = 0;
        Stack<Integer> s = new Stack<>();
        
        while (left < len) {
            while (!s.isEmpty() && h[left] < h[s.peek()]) {
                // int height = h[s.pop()];
                // int width = left - (s.isEmpty ? 0 : s.peek() + 1);
                maxArea = Math.max(maxArea, h[s.pop()] * (left - (s.isEmpty() ? 0 : s.peek() + 1)));
            }
            s.push(left++);
        }
        
        while (!s.isEmpty()) {
            maxArea = Math.max(maxArea, h[s.pop()] * (len - (s.isEmpty() ? 0 : s.peek() + 1)));
        }
        
        return maxArea;
    }
```

# 8. 接雨水

42\. Trapping Rain Water (Hard）

[Leetcode](https://leetcode.com/problems/trapping-rain-water/) / [力扣](https://leetcode-cn.com/problems/trapping-rain-water/)

暴力解法：

```java
    public int trap(int[] h) {
        // 暴力解法：双指针遍历(慢)
        int rain = 0, len = h.length;
        for (int i = 1; i < len - 1; i++) {
            int maxLeft = 0, maxRight = 0;
            // 查找h[i]左边界最大值
            for (int k = i; k >= 0; k--) {
                maxLeft = Math.max(maxLeft, h[k]);
            }
            // 查找h[i]右边界最大值
            for (int j = i; j < len; j++) {
                maxRight = Math.max(maxRight, h[j]);
            }
            rain += Math.min(maxLeft, maxRight) - h[i];
        }
        return rain;
    }
```

单独递减栈：

```java
    public int trap(int[] height) {
        if (height == null || height.length < 2) return 0;
        // 维护一个单独递减栈
        Stack<Integer> stack = new Stack<>();
        int rain = 0, len = height.length;
        for (int i = 0; i < len; i++) {
            // 栈顶元素小于当前元素,弹栈;大于则将当前元素压栈
            while (!stack.isEmpty() && height[stack.peek()] < height[i]) {
                // 有多个相同数值，只留一个
                int cur = stack.pop();
                while (!stack.isEmpty() && height[stack.peek()] == height[cur]) {
                    stack.pop();
                }
                // 计算雨水量
                if (!stack.isEmpty()) {
                    int top = stack.peek();
                    // int w = i - top - 1;
                    // int h = Math.min(height[top] - height[cur], height[i] - height[cur]);
                    rain += Math.min(height[top] - height[cur], height[i] - height[cur]) * (i - top - 1);
                }
            }
            stack.push(i);
        }
        return rain;
    }
```

动态规划：

* 时间复杂度O(n)，空间复杂度O(n)

```java
    public int trap(int[] height) {
        if (height == null || height.length == 0) return 0;
        // leftMax 从左开始的最高柱,rightMax 从右边开始的最高柱
        int res = 0, leftMax = Integer.MIN_VALUE, rightMax = Integer.MIN_VALUE;
        // 双指针扫描数组
        // leftMax和rightMax中间的柱都不影响当前位置可以捕获多少水
        // [4,2,0,3,2,5]
        // left:0 ,right:5  ,leftMax:4 ,rightMax:5, res:0 + 2
        // left:1 ,right:5  ,leftMax:4 ,rightMax:5, res:0 + 2
        // left:2 ,right:5  ,leftMax:4 ,rightMax:5, res:2 + 4
        // left:3 ,right:5  ,leftMax:4 ,rightMax:5, res:6 + 1
        // left:4 ,right:5  ,leftMax:4 ,rightMax:5, res:7 + 2
        // left:5 ,right:5  ,leftMax:4 ,rightMax:5, res:9 + 0
        for (int left = 0, right = height.length - 1; left <= right; ) {
            leftMax = Math.max(leftMax, height[left]);
            rightMax = Math.max(rightMax, height[right]);
            // 木桶原理求得最小高度
            if (leftMax < rightMax) {
                // 注意减去当前位置的柱高度
                res += leftMax - height[left];
                left++;
            } else {
                res += rightMax - height[right];
                right--;
            }
        }
        return res;
    }
```