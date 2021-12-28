<!-- MarkdownTOC -->

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
  - [10. 排序数组](#10-排序数组)
  - [11. 数组中第k大的数字](#11-数组中第k大的数字)
  - [12. 第 N 位数字](#12-第 N 位数字)
- [矩阵](#矩阵)
  - [1. 重塑矩阵](#1-重塑矩阵)
  - [2. 搜索二维矩阵](#2-搜索二维矩阵)
  - [3. 搜索二维矩阵 II](#3-搜索二维矩阵-ii)
  - [4. 有序矩阵的 Kth Element](#4-有序矩阵的-kth-element)
  - [5. 对角元素相等的矩阵](#5-对角元素相等的矩阵)
  - [6. 螺旋矩阵](#6-螺旋矩阵)

<!-- /MarkdownTOC -->
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

[Leetcode](https://leetcode.com/problems/remove-duplicates-from-sorted-nums/) / [力扣](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-nums/)

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

[Leetcode](https://leetcode.com/problems/merge-sorted-nums/) / [力扣](https://leetcode-cn.com/problems/merge-sorted-nums/)

暴力解法：

```java
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        System.numscopy(nums2, 0, nums1, m, n);
        Arrays.sort(nums1);
    }
```

双指针：时间复杂度O(m+n)， **空间复杂度O(1)**

```java
    public void merge(int[] nums1, int m, int[] nums2, int n) {
		int p1 = m - 1, p2 = n - 1, len = m + n - 1;
        
        while (p1 >= 0 && p2 >= 0) {
            // 设置指针 p1 和 p2 分别指向 nums1 和 nums2 的有数字尾部，从尾部值开始比较遍历
            // 同时设置指针 len 指向 nums1 的最末尾，每次遍历比较值大小之后，则进行填充
            nums1[len--] = nums1[p1] > nums2[p2] ? nums1[p1--] : nums2[p2--];
        }
        // 将nums2数组从下标0位置开始，拷贝到nums1数组中，从下标0位置开始，长度为p2+1
        System.numscopy(nums2, 0, nums1, 0, p2 + 1);
    }
```

逆向双指针：时间复杂度O(m+n)， **空间复杂度O(1)**

- 数组nums1已经预留了nums2所有元素的空间，因此可以从数组最后一个元素向前合并，每次比较数组中较大值，放入index所指向位置。
  - 如果指向nums2的指针 p2 < 0，代表nums2中所有元素都已经合并到nums1中，直接退出即可；
  - 指向nums1的指针 p1 < 0，代表nums1 数组中的所有元素重新放置在它们应该处于的位置，*nums2*中剩余的元素就应该依次降序放在*[0,index]*这个区间中。
  - 当两个指针p1,p2 都不 < 0 时，则需要比较nums[i] 和 nums[j]，将较大值放入 nums[index] 中，并且index--，较大值的数组指针 -- 。
  - 最后当指针p1,p2 同时 *< 0* 时，就说明合并完毕直接退出即可。

```java
public void merge(int[] nums1, int m, int[] nums2, int n) {
	int p1 = m - 1, p2 = n - 1, index = m + n - 1;
    
    while (p1 >= 0 || p2 >= 0) {
        if (p2 < 0) {
            break;
        } else if (p1 < 0) {
			nums1[index--] = nums2[p2--];
        } else if (nums1[p1] > nums2[p2]) {
            nums1[index--] = nums1[p1--];
        } else {
            nums1[index--] = nums2[p2--];
        }
    }
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

[Leetcode](https://leetcode.com/problems/rotate-nums/) / [力扣](https://leetcode-cn.com/problems/rotate-nums/)

数组拷贝：

```java
    public void rotate(int[] nums, int k) {
        int[] tmp = new int[nums.length];
        System.numscopy(nums, 0, tmp, 0, nums.length);
        for (int i = 0; i < nums.length; i++) {
            nums[(i + k) % nums.length] = tmp[i];
        }
    }
```

多次反转：空间复杂度O(1)

```java
    public void rotate(int[] nums, int k) {
        if (nums == null || nums.length < 2) return;
        // If the step is two or more times bigger than the input nums length, it equals the remainder.
        // For example, we let the input nums is [1,2,3], and the step is 7, it is as same as 1.
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

------

相关题目：[搜索旋转排列数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

解法：二分查找，时间复杂度`O(logn)`，空间复杂度`O(1)`

- 将数组一分为二，其中一定有段有序，另一段可能有序，也可能部分有序。此时，有序部分二分法查找，无序部分再一分为二，其中一段一定有序，另一段可能有序，可能无序。就这样循环查找，直到找到target为止。

```java
    public int search(int[] nums, int target) {
        int lo = 0, hi = nums.length - 1;
        while (lo <= hi) {
            int mid = lo + (hi - lo >> 1);
            if (target == nums[mid]) return mid;
            // 2,3,4,5,6,7,0,1  mid左段有序
            if (nums[mid] >= nums[lo]) {
                if (target >= nums[lo] && target < nums[mid])
                    hi = mid - 1;
                else
                    lo = mid + 1;
            } else {
                // 6,7,0,1,2,3,4,5  mid右段有序
                if (target > nums[mid] && target <= nums[hi])
                    lo = mid + 1;
                else
                    hi = mid - 1;
            }
        }
        return -1;
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

## 10. 排序数组

912\. Sort An Array(Medium)

[LeetCode](https://leetcode.com/problems/sort-an-nums/) / [力扣](https://leetcode-cn.com/problems/sort-an-nums/)

> 给你一个整数数组 `nums`，请你将该数组升序排列。
>
> 1. `1 <= nums.length <= 50000`
> 2. `-50000 <= nums[i] <= 50000`

**解法一：快速排序**，时间复杂度`O(nlogn)`，空间复杂度`O(logn)`

* 遍历：最常见的实现，选第一个作为Pivot，然后直接从左到右遍历，把数据分割为两部分。

```java
    private static final Random RANDOM = new Random();
   
    public int[] sortArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return nums;
        }

        quickSort(nums, 0, nums.length - 1);
        return nums;
    }

    private void quickSort(int[] nums, int lo, int hi) {
        if (hi <= lo) {
            return;              
        }
       
        int pos = partition(nums, lo, hi);
        // 将左半部分nums[lo ... pos]排序，将右半部分nums[pos+1 ... hi]排序
        quickSort(nums, lo, pos - 1);      
        quickSort(nums, pos + 1, hi);      
    }

    private int partition(int[] nums, int lo, int hi) {
        //  随机选取
        int randomIndex = RANDOM.nextInt(hi - lo + 1) + lo;
        swap(nums, lo, randomIndex);
        
        // 取第一个作为轴
        int pivot = nums[lo];
        // index为pivot最终的位置，最终要满足[lo ~ index-1] < [index] < [index+1 ~ hi]
        int index = lo;
        // 循环过程中，先不移动pivot，满足[lo+1 ~ index] < pivot < [index+1 ~ hi]
        for (int i = lo + 1; i <= hi; i++) {
            if (index <= hi && nums[i] < pivot) {
                index++;
                swap(nums, i, index);
            }
        }
        // 最终交换 lo 和 index，让pivot移动到正确的位置，
        // 使得 [lo ~ index-1] < [index] < [index+1 ~ hi]
        swap(nums, index, lo);
        return index;
    }

    private void swap(int[] nums, int i, int j) {
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
```

* 两端搜索交换：选中最后一个作为Pivot，先从左边找到第一个大于Pivot的元素，再从右边找到第一个小于Pivot的元素，将这两者交换。继续交互步骤直到两端相遇。

```java
    parivte int partition(int[] nums, int lo, int hi) {
        int pivot = nums[lo];

        // 这里有一种特殊情况，
        // 原始数组已经满足 [lo] < [lo+1 ~ hi]，执行循环后，应该得到 i = j = lo

        // i必须从lo开始，而不是 lo + 1
        int i = lo, j = hi;
        while (i < j) {
            // 顺序很重要，要先从右边开始找
            while (i < j && nums[j] >= pivot) --j;
            // 再找左边的
            while (i < j && nums[i] <= pivot) ++i;
            // 交换两个数在数组中的位置
            if (i < j) {
                swap(nums, i, j);
            }
        }
        // i == j情况下，将pivot归位
        swap(nums, lo, i);
        return i;
    }
```

**解法二：桶排序**：时间复杂度O(n)

```java
	public int[] sortArray(int[] nums) {
        // 记录最大最小值
        int min = Integer.MAX_VALUE, max = Integer.MIN_VALUE;
        for (int num : nums) {
            min = Math.min(num, min);
            max = Math.max(num, max);
        }
        int len = max - min;
        //  桶，在值对应的位置上计数
        int bucket[] = new int[len + 1];
        
        for (int num : nums) {
            bucket[num - min]++;
        }
        
        for (int i = 0, j = 0; i <= len && j < nums.length; i++) {
            while (bucket[i]-- != 0) {
                nums[j++] = i + min;
            }
        }
        return nums;
    }
```

**题解三**：堆排序，时间复杂度`O(nlogn)`，空间复杂度`O(1)`

```java
    public int[] sortArray(int[] A) {
        int len = A.length - 1;
        int bigenIndex = (A.length >> 1) - 1;
        // 1.将数组堆化 
        // beginIndex = 第一个非叶子节点。
        for (int i = bigenIndex; i >= 0; i--) {
            maxHeapify(A, i, len);
        }
        // 2.对堆化数据排序
        // 每次都是移出最顶层的根节点A[0]，与最尾部节点位置调换，同时遍历长度 - 1。
        // 然后从新整理被换到根节点的末尾元素，使其符合堆的特性。
        for (int i = len; i > 0 ; i--) {
            swap(A, 0, i);
            maxHeapify(A, 0, i - 1);
        }
        return A;
    }

    private void maxHeapify(int[] A, int i, int len) {
        int lo = (i << 1) + 1; // 左子节点索引
        int hi = lo + 1; // 右子节点索引
        int max = i; // 子节点值最大索引，默认左子节点。

        if (lo > len) return; // 左子节点索引超出计算范围，直接返回。
        if (lo <= len && A[lo] >= A[max]) max = lo;
        if (hi <= len && A[hi] >= A[max]) max = hi;
        if (max != i) {
            // 如果父节点被子节点调换，则需要继续判断换下后的父节点是否符合堆的特性。
            swap(A, i, max);
            maxHeapify(A, max, len);
        }
    }

    private void swap(int[] A, int i, int j) {
        int tmp = A[i];
        A[i] = A[j];
        A[j] = tmp;
    }
```

**题解四**：归并排序，时间复杂度`O(nlogn)`，空间复杂度`O(n)`

```java
class Solution {
    int[] tmp;

    public int[] sortArray(int[] nums) {
        tmp = new int[nums.length];
        mergeSort(nums, 0, nums.length - 1);
        return nums;
    }

    public void mergeSort(int[] nums, int l, int r) {
        if (l >= r) {
            return;
        }
        int mid = (l + r) >> 1;
        mergeSort(nums, l, mid);
        mergeSort(nums, mid + 1, r);
        int i = l, j = mid + 1;
        int cnt = 0;
        
        while (i <= mid && j <= r) {
            if (nums[i] <= nums[j]) tmp[cnt++] = nums[i++];
            else tmp[cnt++] = nums[j++];
        }
        
        while (i <= mid) tmp[cnt++] = nums[i++];
        
        while (j <= r) tmp[cnt++] = nums[j++];
        
        for (int k = 0; k < r - l + 1; ++k) {
            nums[k + l] = tmp[k];
        }
    }
}
```
## 11. 数组中第k大的数字

215\. Kth Largest Element In An Array(Medium)

[LeetCode](https://leetcode.com/problems/kth-largest-element-in-an-nums/)   / [力扣](https://leetcode-cn.com/problems/kth-largest-element-in-an-nums/)  

题目：给定整数数组 `nums` 和整数 `k`，请返回数组中第 `k` 个最大的元素（数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素）

**题解一**：**快速选择算法**，时间复杂度O(n)，空间复杂度O(logn)

- 要找到一个元素X，那么使得小于 X 的元素数量是 k-1 个时，那 X 就是我们要查找的排名第 k 位的元素。那么没必要对数组进行整体排序，找到这个元素 X 即可。

- 如果基准元素的排名 ind 正好等于 k，那说明当前的基准值，就是我们要找的排名第 k 位的元素；
- 如果 ind 大于 k，说明排名第 k 位的元素在基准值的前面；接下来，要解决的问题就是，在基准值的前面查找排名第 k 位的元素。
- 如果 ind 小于 k ，就说明排名第 k 位的元素在基准值的后面，并且，当前包括基准值在内的 ind 个元素，都是小于基准值的元素。那么，问题就转化成了，在基准值的后面查找排名第 k - ind 位的元素。

```java
    private static final Random RANDOM = new Random();

	public int findKthLargest(int[] nums, int k) {
        int lo = 0, hi = nums.length - 1, index = nums.length - k;
        while (lo < hi) {
            int pivot = partion(nums, lo, hi);
            if (pivot < index) {
                lo = pivot + 1; 
            } else if (pivot > index) {
                hi = pivot - 1;
            } else {
                return nums[pivot];
            }
        }
        return nums[lo];
    }
    
    private int partion(int[] nums, int lo, int hi) {
        int randomIndex = RANDOM.nextInt(hi - lo + 1) + lo;
        swap(nums, lo, randomIndex);
        
        int pivot = lo, temp;
        while (lo <= hi) {
            while (lo <= hi && nums[lo] <= nums[pivot]) lo++;
            while (lo <= hi && nums[hi] > nums[pivot]) hi--;
            if (lo > hi) break;
			swap(nums, lo, hi);
        }
        swap(nums, pivot, hi);
        return hi;
    }

    private void swap(int[] nums, int i, int j) {
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
```

**题解二**：堆排序，时间复杂度O(nlogn)，空间复杂度O(nogn)

- 建立一个大根堆，做 k - 1 次删除操作后堆顶元素就是我们要找的答案。

```java
    public int findKthLargest(int[] A, int k) {
        int len = A.length - 1;
        int bigenIndex = (A.length >> 1) - 1;
        // 1.建堆
        for (int i = bigenIndex; i >= 0; i--) {
            maxHeapify(A, i, len);
        }
        // 2.排序
        for (int i = len; i > len - k + 1; i--) {
            swap(A, 0, i);
            maxHeapify(A, 0, i - 1);
        }
        return A[0];
    }

    private void maxHeapify(int[] A, int i, int len) {
        int lo = (i << 1) + 1;
        int hi = lo + 1;
        int max = i;

        if (lo > len) return;
        if (lo <= len && A[lo] >= A[max]) max = lo;
        if (hi <= len && A[hi] >= A[max]) max = hi;

        if (max != i) {
            swap(A, i, max);
            maxHeapify(A, max, len);
        }
    } 

    private void swap(int[] A, int i, int j) {
        int tmp = A[i];
        A[i] = A[j];
        A[j] = tmp;
    }
```

## 12. 第 N 位数字

[力扣](https://leetcode-cn.com/problems/nth-digit/) （Medium）

题目：给你一个整数 n ，请你在无限的整数序列 [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, ...] 中找出并返回第 n 位上的数字。

题解：

```java
    public int findNthDigit(int n) {
      // 基本规律:
      // 第1组：[1, 9]        有9 * 1个数字     => 9 * 10^0 * 1
      // 第2组：[10, 99]      有90 * 2个数字    => 9 * 10^1 * 2
      // 第3组：[100, 999]    有900 * 3个数字   => 9 * 10^2 * 3
      // 第4组：[1000, 9999]  有9000 * 4个数字  => 9 * 10^3 * 4
      // 第k组：                                => 9 * 10^(n-1) * n
      // 1，根据n确定属于第k组
      // 2. (n - 1) / k 来确定是第k组中的第m个数
      // 3. (n - 1) % k 来确定是第m个数中的第x位
      int k = 1;
      while (n > 9 * Math.pow(10, k - 1) * k) {
        n -= 9 * Math.pow(10, k - 1) * k;
        k++;
      }
      int m = (n - 1) / k;
      int x = (n - 1) % k;
      int num = (int)Math.pow(10, k - 1) + m;
      return String.valueOf(num).charAt((int) x) - '0';
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

## 2. 搜索二维矩阵

74\. Search a 2D Matrix  (Medium)

[Leetcode](https://leetcode.com/problems/search-a-2d-matrix/) / [力扣](https://leetcode-cn.com/problems/search-a-2d-matrix/)

二分查找：时间复杂度O(log m*n)，空间复杂度O(1)

- 每行的第一个数要大于前一行最后一个数，因此可将二维数组拼接成一个有序数组，二分查找的关键在于如何根据mid值确定矩阵的两个下标。

```java
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length, n= matrix[0].length;
        if (m == 0) return false;

        int low = 0, high = m * n - 1, mid, value;
        while (lo <= hi) {
            mid = low + (high - low >>> 1);

            value = matrix[mid / n][mid % n];
            if (target == value) return true;
            else {
                if (target < value) hi = mid - 1;
                else lo = mid + 1;
            }
        }
        return false;
    }
```

## 3. 搜索二维矩阵 II

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

## 4. 有序矩阵的 Kth Element

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

## 5. 对角元素相等的矩阵

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



## 6. 螺旋矩阵

[LeetCode](https://leetcode.com/problems/spiral-matrix/) / [力扣](https://leetcode-cn.com/problems/spiral-matrix/)

顺时针螺旋打印矩阵，输入：matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]，输出：[1,2,3,4,8,12,11,10,9,5,6,7]

```java
    public List<Integer> spiralOrder(int[][] matrix) {
      List<Integer> res = new ArrayList<Integer>();
      if (matrix.length == 0 || matrix[0].length == 0) return res;

      int left = 0, right = matrix[0].length - 1;
      int top = 0, bottom = matrix.length - 1;

      while (true) {
        for(int i = left; i <= right; i++) res.add(matrix[top][i]);
        top++;
        if(left > right || top > bottom) break;

        for(int i = top; i <= bottom; i++) res.add(matrix[i][right]);
        right--;
        if(left > right || top > bottom) break;

        for(int i = right; i >= left; i--) res.add(matrix[bottom][i]);
        bottom--;
        if(left > right || top > bottom) break;

        for(int i = bottom; i >= top; i--) res.add(matrix[i][left]);
        left++;
        if(left > right || top > bottom) break;
      }
      return res;
    }
```

