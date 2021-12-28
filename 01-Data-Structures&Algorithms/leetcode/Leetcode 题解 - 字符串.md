<!-- MarkdownTOC -->
- [1. 最长公共子序列](#1-最长公共子序列)
- [2. 最长回文子串](#2-最长回文子串)
- [3. 把数字翻译成字符串](#3-把数字翻译成字符串)
- [4. 字母异位词分组](#4-字母异位词分组)
- [5. 编辑距离](#5-编辑距离)
- [6. 字符串转换整数 (atoi)](#6-字符串转换整数-atoi)
- [7. 括号生成](#7-括号生成)
- [8. 解码方法](#8-解码方法)

<!-- /MarkdownTOC -->

## 1. 最长公共子序列

 [LeetCode](https://leetcode.com/problems/longest-common-subsequence/) /  [力扣](https://leetcode-cn.com/problems/longest-common-subsequence/) （Medium）

题目：给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长 **公共子序列** 的长度。如果不存在 **公共子序列** ，返回 `0` 。例如`text1 = "abcde", text2 = "ace"` ，输出`3`，因为`最长公共子序列是 "ace" ，它的长度为 3` 。

题解：动态规划，时间复杂度`O(m*n)`，空间复杂度`O(m*n)`

* 若当前字符相同，则找到了一个公共元素，并将公共子序列的长度在 `f(i - 1)(j - 1) `的基础上再加 1，此时状态转移方程：`dp[i][j] = dp[i-1][j-1] + 1`。 
* 若当前字符不同，只需要把第一个字符串往前退一个字符或者第二个字符串往前退一个字符然后记录最大值即可（相当于看 `是[0, i - 2] 与 s2[0, j - 1] `的最长公共子序列 和 `s1[0, i - 1] 与 s2[0, j - 2] `的最长公共子序列，取最大的），此时状态转移方程：`dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1])`。

```java
    public int longestCommonSubsequence(String text1, String text2) {
		int m = text1.length(), n = text2.length();
        
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (text1.charAt(i) == text2.charAt(j)) {
                    dp[i + 1][j + 1] = dp[i][j] + 1;
                } else {
                    dp[i + 1][j + 1] = Math.max(dp[i][j + 1], dp[i + 1][j]);
                }
            }
        }
        return dp[m][n];
    }
```



## 2. 最长回文子串

[LeetCode](https://leetcode.com/problems/longest-palindromic-substring/) / [力扣](https://leetcode-cn.com/problems/longest-palindromic-substring/)（Medium）

题目：给你一个字符串 `s`，找到 `s` 中最长的回文子串。例如`s = "babad"`，则输出`"bab"`（`“aba"` 同样OK）

题解：中心扩展法，时间复杂度`O(n^2)`，空间复杂度`O(1)`

```java
    public String longestPalindrome(String s) {
        String res = "";
        for (int i = 0; i < s.length(); i++) {
            String odd = check(s, i, i), even = check(s, i, i + 1);
            if (odd.length() > res.length())
                res = odd;
            if (even.length() > res.length())
                res = even;
        }
        return res;
    }

    public String check(String s, int i, int j) {
        // 双指针向两边扩散
        while (i >= 0 && j < s.length() && s.charAt(i) == s.charAt(j)) {
            i--;
            j++;
        }
        // 返回以i+1和j索引为中心的最⻓回文串
        return s.substring(i + 1, j);
    }
```



## 3. 把数字翻译成字符串

[剑指 Offer. 把数字翻译成字符串](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/) （Medium）

题目：给定一个数字，我们按照如下规则把它翻译为字符串：0 翻译成 “a” ，1 翻译成 “b”，……，11 翻译成 “l”，……，25 翻译成 “z”。一个数字可能有多个翻译。

题解：动态规划，时间复杂度`O(logn)`，空间复杂度`O(logn)`

```java
    public int translateNum(int num) {
      // dp[i] 表示x(i)结尾的数字翻译数量
      // 1. x(i)与x(i-1)能够组合翻译 10*x(i-1) + x(i) -> [10,25]
      //    dp[i] = dp[i-1]+dp[i-2];
      // 2. x(i)与x(i-1)只能单独翻译 10*x(i-1) + x(i) -> [1,10) || (25,99]
      //    dp[i] = dp[i-1];
      // a -> dp[i], b -> dp[i-1],节省O(n)空间
      int a = 1, b = 1, x, y = num % 10;
      while (num != 0) {
        num /= 10;
        x = num % 10;
        int tmp = x * 10 + y;
        int c = (tmp >= 10 && tmp <= 25) ? a + b : a;
        b = a;
        a = c;
        y = x;
      }
      return a;
    }
```



## 4. 字母异位词分组

[LeetCode](https://leetcode.com/problems/group-anagrams/) / [力扣](https://leetcode-cn.com/problems/group-anagrams/) （Medium）

题目：字母异位词 是由重新排列源单词的字母得到的一个新单词，所有源单词中的字母通常恰好只用一次。给你一个字符串数组，请你将 **字母异位词** 组合在一起。可以按任意顺序返回结果列表。

```java
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<String, List<String>>();
        for (String str : strs) {
            char[] array = str.toCharArray();
            Arrays.sort(array);
            String key = new String(array);
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }
        return new ArrayList<List<String>>(map.values());
    }
```



## 5. 编辑距离

[LeetCode](https://leetcode.com/problems/edit-distance/) / [力扣](https://leetcode-cn.com/problems/edit-distance/)（Hard）

```java
    public int minDistance(String word1, String word2) {
      // 1.分治（子问题）
      //  word1长度m, word2长度n, f(m,n)表示word1(0...i) --> word2(0...j)的方案数
      //  (1) m==0, f(m,n)==n || n==0, f(m,n)==m;
      //  (2) 当前字母相同，word1(i) = word2(j),则f(i,j)=f(i-1,j-1)
      //  (3) 当前字母不一样，
      //      插入,word1插入与word2一样的字母，操作数+1，f(i,j)=f(i,j-1)+1
      //      删除,word1删除当前字母，操作数+1，f(i,j)=f(i-1,j)+1
      //      替换,word1当前替换成word2字母，操作数+1，f(i,j)=f(i-1,j-1)+1
      // 2.状态数组定义 dp[m+1][n+1]
      // 3.DP方程
      //  字母相同，dp[i][j] = dp[i-1][j-1];
      //  字母不同，dp[i][j] = Math.min(Math.min(dp[i-1][j-1],dp[i][j-1]), dp[i-1][j])+1;
      int m = word1.length(), n = word2.length();
      // 存在空字符
      if (m * n == 0) return m + n;
      int[][] dp = new int[m + 1][n + 1];
      // 边界条件：第一列
      for (int i = 1; i <= m; i++) {
        dp[i][0] = dp[i - 1][0] + 1; 
      }
      // 边界条件：第一行
      for (int j = 1; j <= n; j++) {
        dp[0][j] = dp[0][j - 1] + 1;
      }
      for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
          if (word1.charAt(i - 1) == word2.charAt(j - 1)) 
            dp[i][j] = dp[i - 1][j - 1];
          else 
            dp[i][j] = Math.min(Math.min(dp[i - 1][j - 1], dp[i][j - 1]), dp[i - 1][j]) + 1;
        }
      }
      return dp[m][n];
    }
```



## 6. 字符串转换整数 (atoi)

[LeetCode](https://leetcode.com/problems/string-to-integer-atoi/) / [力扣](https://leetcode-cn.com/problems/string-to-integer-atoi/)（Medium）

```java
      public int myAtoi(String s) {
        int MAX = Integer.MAX_VALUE, MIN = Integer.MIN_VALUE;
        int idx = 0, res = 0, sign = 1;
        // 1.去前导空格
        while (idx < s.length() && s.charAt(idx) == ' ') {
            idx++;
        }
        // 1.1 极端情况 s == ' '
        if (s.length() == idx) {
            return 0;
        }
        // 2.判断正负
        if (s.charAt(idx) == '+' || s.charAt(idx) == '-') {
            sign = s.charAt(idx) == '+' ? 1 : -1;
            idx++;
         }
        // 3.转换数字
        while (idx < s.length()) {
            int digit = s.charAt(idx) - '0';
            if (digit < 0 || digit > 9) {
                break;
            }
            // 3.1 检测溢出
            if (MAX / 10 < res || MAX / 10 == res && MAX % 10 < digit) {
                return sign == 1 ? MAX : MIN;
            }
            // 3.2 计算值
            res = res * 10 + digit;
            idx++;
        }
        return res * sign;
      }
```



## 7. 括号生成

[LeetCode](https://leetcode.com/problems/generate-parentheses/) / [力扣](https://leetcode-cn.com/problems/generate-parentheses/)（Medium）

```java
    List<String> res = new ArrayList<>();
    public List<String> generateParenthesis(int n) {
      String cur = "";
      dfs(0, 0, n, cur);
      return res;
    }

    private void dfs(int le, int ri, int n, String str) {
      if (le == n && ri == n) {// 左右括号数量相等，产生一个满足条件的值
        res.add(str);
        return;
      }
      if (le < 0 || ri < 0 || le < ri) return;// 剪枝
      if (le < n) dfs(le + 1, ri, n, str + "(");// 产生左分支
      if (le > ri) dfs(le, ri + 1, n, str + ")");// 只有右括号小于左括号数量，产生右分支
    }
```



## 8. 解码方法

[LeetCode](https://leetcode.com/problems/decode-ways/) / [力扣](https://leetcode-cn.com/problems/decode-ways/)（Medium）

```java
  public int numDecodings(String s) {
    int n = s.length();
    if (n == 0) return 0;

    int[] memo = new int[n + 1];
    memo[n] = 1;
    memo[n - 1] = s.charAt(n - 1) != '0' ? 1 : 0;

    for (int i = n - 2; i >= 0; i--) {
      if (s.charAt(i) == '0') continue;
      else memo[i] = (Integer.parseInt(s.substring(i, i + 2)) <= 26) ? memo[i + 1] + memo[i + 2] : memo[i + 1];
    }
    return memo[0];
  }
```

