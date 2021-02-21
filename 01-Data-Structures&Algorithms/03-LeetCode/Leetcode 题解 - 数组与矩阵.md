- [数组](#数组)
  - [1. 移动零](#1-移动零)
  - [2. 最大连续 1 的个数](#2-最大连续-1-的个数)
  - [3. 加一](#3-加一)
  - [4. 删除排序数组中的重复项](#4-删除排序数组中的重复项)
  - [5. 合并两个有序数组](#5-合并两个有序数组)
  - [6. 盛最多水的容器](#6-盛最多水的容器)
  - [7. 旋转数组](#7-旋转数组)
  - [8. 三数之和](#8-三数之和)
  - [9. 有效的数独](#9-有效的数独)
- [矩阵](#矩阵)
  - [1. 改变矩阵维度](#1-改变矩阵维度)
  - [2. 有序矩阵查找](#2-有序矩阵查找)
  - [3. 有序矩阵的 Kth Element](#3-有序矩阵的-kth-element)
  - [4. 对角元素相等的矩阵](#4-对角元素相等的矩阵)
# 数组

## 1. 移动零

283\. Move Zeroes (Easy)

[Leetcode](https://leetcode.com/problems/move-zeroes/description/) / [力扣](https://leetcode-cn.com/problems/move-zeroes/description/)

```html
For example, given nums = [0, 1, 0, 3, 12], after calling your function, nums should be [1, 3, 12, 0, 0].
```

```java
public void moveZeroes(int[] nums) {
    int idx = 0;
    for (int num : nums) {
        if (num != 0) {
            nums[idx++] = num;
        }
    }
    while (idx < nums.length) {
        nums[idx++] = 0;
    }
}
```

## 2. 最大连续 1 的个数

485\. Max Consecutive Ones (Easy)

[Leetcode](https://leetcode.com/problems/max-consecutive-ones/description/) / [力扣](https://leetcode-cn.com/problems/max-consecutive-ones/description/)

```java
public int findMaxConsecutiveOnes(int[] nums) {
    int cur = 0, max = 0;
    for (int n : nums)
        max = Math.max(max, cur = n == 0 ? 0 : cur + 1);
    return max;     
}
```

## 3. 加一

66\. Plus One（Easy）

[Leetcode](https://leetcode.com/problems/plus-one/) / [力扣](https://leetcode-cn.com/problems/plus-one/)

```java
    public int[] plusOne(int[] digits) {
        for (int i = digits.length - 1; i >= 0; i--) {
            if (digits[i] < 9) {
                digits[i]++;
                return digits;
            }
            digits[i] = 0;
        }
        digits = new int[digits.length + 1];
        digits[0] = 1;
        return digits;
    }
```

## 4. 删除排序数组中的重复项

26\. Remove Duplicates from Sorted Array（Easy）

[Leetcode](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) / [力扣](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

```java
     /**
     * 双指针法
     * 数组有序，重复的元素一定会相邻。
     */
	public int removeDuplicates(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        int p = 0, q = 1;

        while (q < nums.length) {
            if (nums[p] != nums[q]) {
                nums[p + 1] = nums[q];
                p++;
            }
            q++;
        }
        return p + 1;
    }
```

## 5. 合并两个有序数组

88\. Merge Sorted Array（Easy）

[Leetcode](https://leetcode.com/problems/merge-sorted-array/) / [力扣](https://leetcode-cn.com/problems/merge-sorted-array/)

暴力解法：

```java
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        System.arraycopy(nums2, 0, nums1, m, n);
        Arrays.sort(nums1);
    }
```

双指针：

```java
    public void merge(int[] nums1, int m, int[] nums2, int n) {
		int p1 = m - 1, p2 = n - 1, len = m + n - 1;
        
        while (p1 >= 0 && p2 >= 0) {
            // 设置指针 p1 和 p2 分别指向 nums1 和 nums2 的有数字尾部，从尾部值开始比较遍历
            // 同时设置指针 len 指向 nums1 的最末尾，每次遍历比较值大小之后，则进行填充
            nums1[len--] = nums1[p1] > nums2[p2] ? nums1[p1--] : nums2[p2--];
        }
        // 将nums2数组从下标0位置开始，拷贝到nums1数组中，从下标0位置开始，长度为p2+1
        System.arraycopy(nums2, 0, nums1, 0, p2 + 1);
    }
```

## 6. 盛最多水的容器

11\. Container With Most Water(Medium)

[Leetcode](https://leetcode.com/problems/container-with-most-water/) / [力扣](https://leetcode-cn.com/problems/container-with-most-water/)

双指针法：

```java
    public int maxArea(int[] height) {
        int i = 0, j = height.length - 1, water = 0;

        while (i < j) {
            int h = Math.min(height[i], height[j]);
            water = Math.max(water, (j - i) * h);
            while (i < j && height[i] <= h) i++;
            while (i < j && height[j] <= h) j--;
        }
        return water;
    }
```

## 7. 旋转数组

189\. Rotate Array（Meduim）

[Leetcode](https://leetcode.com/problems/rotate-array/) / [力扣](https://leetcode-cn.com/problems/rotate-array/)

数组拷贝：

```java
    public void rotate(int[] nums, int k) {
        int[] tmp = new int[nums.length];
        System.arraycopy(nums, 0, tmp, 0, nums.length);
        for (int i = 0; i < nums.length; i++) {
            nums[(i + k) % nums.length] = tmp[i];
        }
    }
```

多次反转：空间复杂度O(1)

```java
    public void rotate(int[] nums, int k) {
        if (nums == null || nums.length < 2) return;
        // If the step is two or more times bigger than the input array length, it equals the remainder.
        // For example, we let the input array is [1,2,3], and the step is 7, it is as same as 1.
        k = k % nums.length;
        reverse(nums, 0, nums.length - k - 1);
        reverse(nums, nums.length - k, nums.length - 1);
        reverse(nums, 0, nums.length - 1);
    }

    public void reverse(int[] nums, int i, int j) {
        int tmp = 0;
        while (i < j) {
            tmp = nums[i];
            nums[i] = nums[j];
            nums[j] = tmp;
            i++;
            j--;
        }
    }
```

## 8. 三数之和

15\. 3Sum (Medium)

[Leetcode](https://leetcode.com/problems/3sum/) / [力扣](https://leetcode-cn.com/problems/3sum/)

1\. 三重遍历：O(n^3^)

2\. **排序+双指针**：O(n^2^)

* 对数组进行排序。

* 对于数组长度N，如果数组为 null 或者 数组长度小于 3，直接返回 []。

* 遍历排序后数组：

  * 若 nums[i]>0 ：三数和不可能为0，直接返回结果。

  * 对于重复元素：跳过，避免出现重复解。
  * 令左指针 L= i+1，右指针 R=N-1，当L<R 时，执行循环：
    * 当` nums[i]+nums[L]+nums[R]==0`，执行循环，判断左界和右界是否和下一位置重复，去除重复解。并同时将L,R 移到下一位置，寻找新的解。
    * 若和大于 0，说明 nums[R] 太大，R左移
    * 若和小于 0，说明 nums[L] 太小，L右移

```java
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> res = new ArrayList<>();
        if (nums == null || nums.length < 3) return res;

        for (int i = 0; i < nums.length - 1; i++) {
            if (nums[i] > 0) break;
			if (i > 0 && nums[i] == nums[i - 1]) continue;
            int L = i + 1, R = nums.length - 1;
            while (L < R) {
				int sum = nums[i] + nums[L] + nums[R];
                if (sum == 0) {
					res.add(Arrays.asList(nums[i], nums[L], nums[R]));
                    while (L < R && nums[L] == nums[L + 1]) L++;
                    while (L < R && nums[R] == nums[R - 1]) R--;
                    L++;
                    R--;
                } 
                else if (sum < 0) L++;
                else R--;
            }
        }
        return res;
    }
```

## 9. 有效的数独

[Leetcode](https://leetcode.com/problems/valid-sudoku/) / [力扣](https://leetcode-cn.com/problems/valid-sudoku/)

```
判断一个 9x9 的数独是否有效。
输入:
[
  ["5","3",".",".","7",".",".",".","."],
  ["6",".",".","1","9","5",".",".","."],
  [".","9","8",".",".",".",".","6","."],
  ["8",".",".",".","6",".",".",".","3"],
  ["4",".",".","8",".","3",".",".","1"],
  ["7",".",".",".","2",".",".",".","6"],
  [".","6",".",".",".",".","2","8","."],
  [".",".",".","4","1","9",".",".","5"],
  [".",".",".",".","8",".",".","7","9"]
]
输出: true
```

```java
    public boolean isValidSudoku(char[][] board) {
        boolean[][] row = new boolean[9][9], col = new boolean[9][9], area = new boolean[9][9];
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                if (board[i][j] == '.') continue;
                int cur = board[i][j] - '1', idx = i / 3 * 3 + j / 3;
                // 行、列、3*3数独块中没有重复数字
                if (!row[i][cur] && !col[j][cur] && !area[idx][cur])
                    row[i][cur] = col[j][cur] = area[idx][cur] = true;
                else
                    return false;
            }
        }
        return true;
    }
```

# 矩阵

## 1. 重塑矩阵

566\. Reshape the Matrix (Easy)

[Leetcode](https://leetcode.com/problems/reshape-the-matrix/description/) / [力扣](https://leetcode-cn.com/problems/reshape-the-matrix/description/)

```html
Input:
nums =
[[1,2],
 [3,4]]
r = 1, c = 4

Output:
[[1,2,3,4]]

Explanation:
The row-traversing of nums is [1,2,3,4]. The new reshaped matrix is a 1 * 4 matrix, fill it row by row by using the previous list.
```

二维数组的一维表示：

* 将二维数组nums 映射成一个一维数组（i =  x / n, j = x % n）;
* 将这个一维数组映射回 r行 c列的二维数组。

```java
    public int[][] matrixReshape(int[][] nums, int r, int c) {
        int m = nums.length, n = nums[0].length;
        if (m * n != r * c) return nums;

        int[][] res = new int[r][c];
        int index = 0;
        for (int i = 0; i < r; i++) {
            for (int j = 0; j < c; j++) {
                res[i][j] = nums[index / n][index % n];
                index++;
            }
        }
//        for (int x = 0; x < m * n; ++x) {
//            res[x / c][x % c] = nums[x / n][x % n];
//        }        
        return res;
    }
```

## 2. 搜索二维矩阵 II

240\. Search a 2D Matrix II (Medium)

[Leetcode](https://leetcode.com/problems/search-a-2d-matrix-ii/description/) / [力扣](https://leetcode-cn.com/problems/search-a-2d-matrix-ii/description/)

```html
[
   [ 1,  5,  9],
   [10, 11, 13],
   [12, 13, 15]
]
```

暴力解法 O(n^2^)：

```java
    public boolean searchMatrix2(int[][] matrix, int target) {
        for (int i = 0; i < matrix.length; i++) {
            for (int j = 0; j < matrix[0].length; j++) {
                if (matrix[i][j] == target) return true;
            }
        }
        return false;
    }
```

“折线搜索”：由于矩阵的每行的元素从左到右升序排列、每列的元素从上到下升序排列。那么我们从左下角开始搜索矩阵，将当前位置初始化到左下角，如果目标小于当前位置中的值，则目标不能位于当前位置的整个行中；如果目标大于当前位置的值，则目标不能在当前列中。 这样我们可以每次排除一排或一列，因此时间复杂度为O(m + n)。

```java
    public boolean searchMatrix(int[][] matrix, int target) {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) return false;

        int row = 0, col = matrix[0].length - 1;
        while (col >= 0 && row <= matrix.length - 1) {
            if (target == matrix[row][col]) return true;
            else if (target < matrix[row][col]) col--;
            else row++;
        }
        return false;
    }
```

## 3. 有序矩阵的 Kth Element

378\. Kth Smallest Element in a Sorted Matrix ((Medium))

[Leetcode](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/description/) / [力扣](https://leetcode-cn.com/problems/kth-smallest-element-in-a-sorted-matrix/description/)

```html
matrix = [
  [ 1,  5,  9],
  [10, 11, 13],
  [12, 13, 15]
],
k = 8,

return 13.
```

直接查找：

* 时间复杂度O(n^2^logn)，空间复杂度O(n^2^)

```java
    public int kthSmallest(int[][] matrix, int k) {
        int n = matrix.length;
        int[] sorted = new int[n * n];
        int index = 0;
        for (int[] row : matrix) {
            for (int num : row) {
                sorted[index++] = num;
            }
        }
        Arrays.sort(sorted);
        return sorted[k - 1];
    }
```

二分查找解法：

* 时间复杂度O(n)，空间复杂度O(1)

```java
    public int kthSmallest(int[][] matrix, int k) {
        int n = matrix.length;
        int lo = matrix[0][0], hi = matrix[n - 1][n - 1];
        while (lo <= hi) {
            int mid = lo + (hi - lo >>> 1);
            int count = getLessEqual(matrix, mid);

            if (count < k) lo = mid + 1;
            else hi = mid - 1;
        }
        return lo;
    }

    private int getLessEqual(int[][] matrix, int val) {
        int res = 0;
        int n = matrix.length, i = n - 1, j = 0;
        while (i >= 0 && j < n) {
            if (matrix[i][j] > val) i--;
            else {
                res += i + 1;
                j++;
            }
        }
        return res;
    }
```

堆解法：

```java
public int kthSmallest(int[][] matrix, int k) {
    int m = matrix.length, n = matrix[0].length;
    PriorityQueue<Tuple> pq = new PriorityQueue<Tuple>();
    for(int j = 0; j < n; j++) pq.offer(new Tuple(0, j, matrix[0][j]));
    for(int i = 0; i < k - 1; i++) { // 小根堆，去掉 k - 1 个堆顶元素，此时堆顶元素就是第 k 的数
        Tuple t = pq.poll();
        if(t.x == m - 1) continue;
        pq.offer(new Tuple(t.x + 1, t.y, matrix[t.x + 1][t.y]));
    }
    return pq.poll().val;
}

class Tuple implements Comparable<Tuple> {
    int x, y, val;
    public Tuple(int x, int y, int val) {
        this.x = x; this.y = y; this.val = val;
    }

    @Override
    public int compareTo(Tuple that) {
        return this.val - that.val;
    }
}
```

## 4. 对角元素相等的矩阵

766\. Toeplitz Matrix (Easy)

[Leetcode](https://leetcode.com/problems/toeplitz-matrix/description/) / [力扣](https://leetcode-cn.com/problems/toeplitz-matrix/description/)

```html
1234
5123
9512

In the above grid, the diagonals are "[9]", "[5, 5]", "[1, 1, 1]", "[2, 2, 2]", "[3, 3]", "[4]", and in each diagonal all elements are the same, so the answer is True.
```

暴力解法：

* 时间复杂度O(m*n)

```java
	public boolean isToeplitzMatrix(int[][] matrix) {
        for (int i = 0; i < matrix.length; i++) {
            for (int j = 0; j < matrix[0].length; j++) {
                if (i > 0 && j > 0 && matrix[i - 1][j - 1] != matrix[i][j]) {
                    return false;
                }
            }
        }
        return true;
    }
```
