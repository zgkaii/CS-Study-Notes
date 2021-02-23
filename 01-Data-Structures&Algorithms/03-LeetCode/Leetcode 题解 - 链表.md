# 目录

- [1. 找出两个链表的交点](#1-找出两个链表的交点)
- [2. 链表反转](#2-链表反转)
- [3. 归并两个有序的链表](#3-归并两个有序的链表)
- [4. 环形链表](#4-环形链表)
- [5. 回文链表](#5-回文链表)
- [6. 从有序链表中删除重复节点](#6-从有序链表中删除重复节点)
- [7. 删除链表的倒数第 n 个节点](#7-删除链表的倒数第-n-个节点)
- [8. 交换链表中的相邻结点](#8-交换链表中的相邻结点)
- [9. 两数相加 II](#9-两数相加-ii)
- [10. 分隔链表](#10-分隔链表)
- [11. 链表元素按奇偶聚集](#11-链表元素按奇偶聚集)
- [12. 环形链表 II](#12-环形链表-ii)
- [13. LRU 缓存机制](#13-lru-缓存机制)
- [14. 复制带随机指针的链表](#14-复制带随机指针的链表)
- [15. K 个一组翻转链表](#15-k-个一组翻转链表)
#  1. 找出两个链表的交点

160\. Intersection of Two Linked Lists (Easy)

[Leetcode](https://leetcode.com/problems/intersection-of-two-linked-lists/description/) / [力扣](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/description/)

设 A 的长度为 a + c，B 的长度为 b + c，其中 c 为尾部公共部分长度，可知 a + c + b = b + c + a。

当访问 A 链表的指针访问到链表尾部时，令它从链表 B 的头部开始访问链表 B；同样地，当访问 B 链表的指针访问到链表尾部时，令它从链表 A 的头部开始访问链表 A。这样就能控制访问 A 和 B 两个链表的指针能同时访问到交点。

> 两者速度一致，相同时间走的路程一致，那么会同一时间到达终点。

如果不存在交点，那么 a + b = b + a，以下实现代码中 l1 和 l2 会同时为 null，从而退出循环。

```java
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) return null;

        ListNode p1 = headA, p2 = headB;
        while (p1 != p2) {
            p1 = (p1 == null) ? headB : p1.next;
            p2 = (p2 == null) ? headA : p2.next;
        }
        return p1;
    }
```

#  2. 链表反转

206\. Reverse Linked List (Easy)

[Leetcode](https://leetcode.com/problems/reverse-linked-list/description/) / [力扣](https://leetcode-cn.com/problems/reverse-linked-list/description/)

递归：

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;
    
    ListNode next = head.next;
    ListNode newHead = reverseList(next);// 从当前节点的下一个结点开始递归调用。
    next.next = head;// head挂到next节点的后面就完成了链表的反转。
    head.next = null;// 这里head相当于变成了尾结点,尾结点都是为空的,否则会构成环。
    
    return newHead;
}
```

头插法：

```java
public ListNode reverseList(ListNode head) {
    ListNode newHead = new ListNode(-1);
    while (head != null) {
        ListNode next = head.next;
        head.next = newHead.next;
        newHead.next = head;
        head = next;
    }
    return newHead.next;
}
```

双指针：

```java
    public ListNode reverseList(ListNode head) {
        ListNode cur = head, end = null;
        
        while (cur != null) {
            ListNode tmp = cur.next;// 每次访问的原链表节点都会成为新链表的头结点
            cur.next = end;
            end = cur;// 更新新链表
            cur = tmp;
        }
        return end;
    }
```

#  3. 归并两个有序的链表

21\. Merge Two Sorted Lists (Easy)

[Leetcode](https://leetcode.com/problems/merge-two-sorted-lists/description/) / [力扣](https://leetcode-cn.com/problems/merge-two-sorted-lists/description/)

递归：

```java
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
		if(l1 == null) return l2;
		if(l2 == null) return l1;
		if(l1.val < l2.val){
			l1.next = mergeTwoLists(l1.next, l2);
			return l1;
		} else{
			l2.next = mergeTwoLists(l1, l2.next);
			return l2;
		}
    }
```

迭代：

```java
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l2 == null) return l1;

        ListNode dummy = new ListNode(0);
        ListNode cur = dummy;
        while(l1 != null && l2!= null){
            if(l1.val <= l2.val){
                cur.next = l1;
                l1 = l1.next;
            }else {
                cur.next = l2;
                l2 = l2.next;
            }
            cur = cur.next;
        }
        cur.next = (l1 == null) ? l2 : l1;
        return dummy.next;
    }
```

# 4. 环形链表

141\. Linked List Cycle（Easy）

[Leetcode](https://leetcode.com/problems/linked-list-cycle/) / [力扣](https://leetcode-cn.com/problems/linked-list-cycle/)

```java
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) return false;
        ListNode slow = head, fast = head.next;
        
        while (slow != fast) {
            if (fast == null || fast.next == null) return false;
            slow = slow.next;
            fast = fast.next.next;
        }
        return true;
    }
```

#  5. 回文链表

234\. Palindrome Linked List (Easy)

[Leetcode](https://leetcode.com/problems/palindrome-linked-list/description/) / [力扣](https://leetcode-cn.com/problems/palindrome-linked-list/description/)

题目要求：以 **O(1)** 的空间复杂度来求解。

“快慢指针“：

* 找到前半部分链表的尾节点。
* 反转后半部分链表。
* 判断是否回文。
* 恢复链表。
* 返回结果。

```java
	public boolean isPalindrome2(ListNode head) {
        // 1 ->2->2->1->null => 1->2->2->1->null => 1->2  null<-2<-1  (偶数节点)
        // s/f                        s     f       h              s
        // 1 ->2->3->2->1->null => 1->2->3->2->1->null => 1->2 3<-2<-1 (奇数节点)
        // s/f                           s        f       h          s
        ListNode fast = head, slow = head;
        while (fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }
        if (fast != null) slow = slow.next;

        slow = reverse(slow);
        while (slow != null && head.val == slow.val) {
            head = head.next;
            slow = slow.next;
        }
        return slow == null;
    }

    private ListNode reverse(ListNode head) {
        ListNode prev = null;
        while (head != null) {
            ListNode next = head.next;
            head.next = prev;
            prev = head;
            head = next;
        }
        return prev;
    }
```

#  6. 从有序链表中删除重复节点

83\. Remove Duplicates from Sorted List (Easy)

[Leetcode](https://leetcode.com/problems/remove-duplicates-from-sorted-list/description/) / [力扣](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/description/)

```html
Given 1->1->2, return 1->2.
Given 1->1->2->3->3, return 1->2->3.
```

递归：

```java
public ListNode deleteDuplicates(ListNode head) {
    if (head == null || head.next == null) return head;
    head.next = deleteDuplicates(head.next);
    return head.val == head.next.val ? head.next : head;
}
```

迭代：

```java
    public ListNode deleteDuplicates(ListNode head) {
         ListNode cur = head;
        
         while(cur != null) {
        	 if (cur.next == null) break;
        	 if (cur.val == cur.next.val) {
        		 cur.next = cur.next.next;
        	 } else {
        		 cur = cur.next;
        	 }
         }
         return head;
    }
```

#  7. 删除链表的倒数第 n 个节点

19\. Remove Nth Node From End of List (Medium)

[Leetcode](https://leetcode.com/problems/remove-nth-node-from-end-of-list/description/) / [力扣](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/description/)

```html
Given linked list: 1->2->3->4->5, and n = 2.
After removing the second node from the end, the linked list becomes 1->2->3->5.
```

"快慢指针"：

```java
    public ListNode removeNthFromEnd(ListNode head, int n) {
        // 1,2,3,4,5,null  =>  1,2,3,4,5,null   n = 2
        // p     q                 p       q
        ListNode p = head, q = head;
        while (n-- > 0) q = q.next;
        if(q == null) return q.next;
        while (q.next != null) {
            q = q.next;
            p = p.next;
        }
        // 1,2,3,5,null
        p.next = p.next.next;
        return head;
    }
```

#  8. 交换链表中的相邻结点

24\. Swap Nodes in Pairs (Medium)

[Leetcode](https://leetcode.com/problems/swap-nodes-in-pairs/description/) / [力扣](https://leetcode-cn.com/problems/swap-nodes-in-pairs/description/)

```html
Given 1->2->3->4, you should return the list as 2->1->4->3.
```

题目要求：不能修改结点的 val 值，O(1) 空间复杂度。

递归：

* 时间复杂度O(n)，空间复杂度O(n)


```java
    public ListNode swapPairs(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode newHead = head.next;
        head.next = swapPairs(newHead.next);
        newHead.next = head;
        return newHead;
    }
```

迭代：

* 时间复杂度O(n)，空间复杂度O(1)

```java
	public ListNode swapPairs(ListNode head) {
        ListNode dummyHead = new ListNode(0);
        dummyHead.next = head;
        ListNode temp = dummyHead;
        while (temp.next != null && temp.next.next != null) {
            ListNode node1 = temp.next;
            ListNode node2 = temp.next.next;
            temp.next = node2;
            node1.next = node2.next;
            node2.next = node1;
            temp = node1;
        }
        return dummyHead.next;
	}
```

#  9. 两数相加 II

445\. Add Two Numbers II (Medium)

[Leetcode](https://leetcode.com/problems/add-two-numbers-ii/description/) / [力扣](https://leetcode-cn.com/problems/add-two-numbers-ii/description/)

```html
Input: (7 -> 2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 8 -> 0 -> 7
```

题目要求：不能修改原始链表。

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    Stack<Integer> l1Stack = buildStack(l1);
    Stack<Integer> l2Stack = buildStack(l2);
    ListNode head = new ListNode(-1);
    int carry = 0;
    while (!l1Stack.isEmpty() || !l2Stack.isEmpty() || carry != 0) {
        int x = l1Stack.isEmpty() ? 0 : l1Stack.pop();
        int y = l2Stack.isEmpty() ? 0 : l2Stack.pop();
        int sum = x + y + carry;
        ListNode node = new ListNode(sum % 10);
        node.next = head.next;
        head.next = node;
        carry = sum / 10;
    }
    return head.next;
}

private Stack<Integer> buildStack(ListNode l) {
    Stack<Integer> stack = new Stack<>();
    while (l != null) {
        stack.push(l.val);
        l = l.next;
    }
    return stack;
}
```

#  10. 分隔链表

725\. Split Linked List in Parts(Medium)

[Leetcode](https://leetcode.com/problems/split-linked-list-in-parts/description/) / [力扣](https://leetcode-cn.com/problems/split-linked-list-in-parts/description/)

题目要求：把链表分隔成 k 部分，每部分的长度都应该尽可能相同，排在前面的长度应该大于等于后面的。

```java
    public ListNode[] splitListToParts(ListNode root, int k) {
        ListNode[] parts = new ListNode[k];
        int len = 0;
        for (ListNode node = root; node != null; node = node.next)
            len++;
        int n = len / k, r = len % k; // n : minimum guaranteed part size; r : extra nodes spread to the first r parts;
        ListNode node = root, prev = null;
        for (int i = 0; node != null && i < k; i++, r--) {
            parts[i] = node;
            for (int j = 0; j < n + (r > 0 ? 1 : 0); j++) {
                prev = node;
                node = node.next;
            }
            prev.next = null;
        }
        return parts;        
    }
```

#  11. 链表元素按奇偶聚集

328\. Odd Even Linked List (Medium)

[Leetcode](https://leetcode.com/problems/odd-even-linked-list/description/) / [力扣](https://leetcode-cn.com/problems/odd-even-linked-list/description/)

```html
Example:
Given 1->2->3->4->5->NULL,
return 1->3->5->2->4->NULL.
```

题目要求：算法的空间复杂度应为 O(1)，时间复杂度应为 O(n)，n为节点总数。

```java
    public ListNode oddEvenList(ListNode head) {
        // 按奇偶分离链表，再合并链表
        // 1->2->3->4->5->null => 1->3->5  2->4->null => 1->3->5->2->4->null
        if(head == null || head.next == null) return head;
        ListNode odd = head, even = head.next, evenHead = even;
        
        while(odd.next != null && even.next != null){
            odd.next = even.next;
            odd = odd.next;
            even.next = odd.next;
            even = even.next;
        }
        odd.next = evenHead;
        return head;
    }
```

# 12. 环形链表 II

142\. Linked List Cycle II (Medium)

[Leetcode](https://leetcode.com/problems/linked-list-cycle-ii/) / [力扣](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

题目进阶：空间复杂度O(1)

```java
    public ListNode detectCycle(ListNode head) {
        if (head == null || head.next == null) return null;
        // 双指针:速度fast = 2*slow  同时从head出发，设入口点为p,相遇点为w
        //      |p       
        // 3 -> 2 -> 0 -> 4 -> 1
        //       \            /
        //         4 <- 6 <- 5  
        //         |w         
        // 距离: head->p: A, p->w: B, w->p: C
        // slow : A + B, fast = A + 2B + C => A = C
        // 所以在相遇后,slow继续运行,从与fast的相遇点到环入口点(C),另一个 slow2 刚好与slow在环入口点相遇(A)
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            while (slow == fast) {
                ListNode slow2 = head;
                while (slow != slow2) {
                    slow = slow.next;
                    slow2 = slow2.next;
                }
                return slow;
            }
        }
        return null;
    }
```

# 13. LRU 缓存机制

146\. LRU Cache(Medium)

[Leetcode](https://leetcode.com/problems/lru-cache/) / [力扣](https://leetcode-cn.com/problems/lru-cache/)

```java
public class LRUCache {
    private int capacity;
    private int size;
    private DLinkedNode head, tail;
    private Map<Integer, DLinkedNode> cache = new HashMap<>();

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.size = 0;
        // 伪头部和伪尾部标记界限
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        DLinkedNode node = cache.get(key);
        if (node == null) {
            return -1;
        }
        // key存在，先通过哈希表定位，再移到头部
        moveToHead(node);
        return node.value;
    }

    public void put(int key, int value) {
        DLinkedNode node = cache.get(key);
        if (node == null) {
            // key不存在，双链表头部创建新节点
            DLinkedNode newNode = new DLinkedNode(key, value);
            cache.put(key, newNode);
            addToHead(newNode);
            ++size;
            // 超容，删除双链表尾节点，删除哈希表中对应项
            if (size > capacity) {
                DLinkedNode tail = removeTail();
                cache.remove(tail.key);
                --size;
            }
            // 哈希表定位，修改value，移到头部
        } else {
            node.value = value;
            moveToHead(node);
        }
    }

    private void addToHead(DLinkedNode node) {
        node.prev = head;
        node.next = head.next;
        node.next.prev = node;
        head.next = node;
    }

    private void removeNode(DLinkedNode node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void moveToHead(DLinkedNode node) {
        removeNode(node);
        addToHead(node);
    }

    private DLinkedNode removeTail() {
        DLinkedNode res = tail.prev;
        removeNode(res);
        return res;
    }
}

class DLinkedNode {
    public int key;
    public int value;

    DLinkedNode prev;
    DLinkedNode next;

    public DLinkedNode() {
    }

    public DLinkedNode(int key, int value) {
        this.key = key;
        this.value = value;
    }
}
```

# 14. 复制带随机指针的链表

138\. Copy List with Random Pointer（Medium）

[Leetcode](https://leetcode.com/problems/copy-list-with-random-pointer/) / [力扣](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)

回溯：

* 时间复杂度O(n) 空间复杂度O(n)

```java
    public Node copyRandomList1(Node head) {
        if (head == null) return null;
        // 如果我们已经处理了当前节点，那么我们只需返回它的克隆版本。
        if (this.visited.containsKey(head)) {
            return this.visited.get(head);
        }
        // 创建一个值与旧节点相同的新节点(即复制节点)。
        Node node = new Node(head.val, null, null);

        // 将此值保存在HashMap中。因为遍历过程中可能由于随机指针的随机性而导致循环,这里将有助于我们避免循环。
        this.visited.put(head, node);

        // 从下一个指针开始递归复制剩余的链表,然后从随机指针开始递归复制。
        // 因此我们有两个独立的递归调用,最后我们为创建的新节点更新了下一个指针和随机指针。
        node.next = this.copyRandomList1(head.next);
        node.random = this.copyRandomList1(head.random);
        return node;
    }
```

进阶：

* 时间复杂度O(n) 空间复杂度O(1)

```java
    public Node copyRandomList(Node head) {
        if (head == null) return null;
        //        —————————                        —————————————————————             ———————————
        //       |         |                      |                     |           |           |
        //  1 -> 2 -> 3 -> 4   => 1 -> 1‘ -> 2 -> 2’ -> 3 -> 3‘ -> 4 -> 4’ => 1' -> 2' -> 3' -> 4'
        //  |         |                |                     |                |           |
        //   —————————                  —————————————————————                  ———————————
        // 第一步：将每个拷贝节点都放在原来对应节点的旁边。
        Node iter = head, next;
        while (iter != null) {
            next = iter.next;
            Node copy = new Node(iter.val);

            iter.next = copy;
            copy.next = next;
            iter = next;
        }

        // 第二步：为拷贝节点分配随机指针。
        iter = head;
        while (iter != null) {
            if (iter.random != null) {
                iter.next.random = iter.random.next;
            }
            iter = iter.next.next;
        }

        // 第三步：恢复原始链表,提取克隆链表。
        iter = head;
        Node pseudoHead = new Node(0);
        Node copy, copyIter = pseudoHead;

        while (iter != null) {
            next = iter.next.next;
            // 提取克隆链表
            copy = iter.next;
            copyIter.next = copy;
            copyIter = copy;
            // 保存原始链表
            iter.next = next;

            iter = next;
        }
        return pseudoHead.next;
    }

    class Node {
        int val;
        Node next;
        Node random;

        public Node(int val, Node next, Node random) {
            this.val = val;
            this.next = next;
            this.random = random;
        }

        public Node(int val) {
            this.val = val;
        }
    }
```

# 15. K 个一组翻转链表

25\. Reverse Nodes in k-Group（Hard）

[Leetcode](https://leetcode.com/problems/reverse-nodes-in-k-group/) / [力扣](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

题目要求：使用常数的额外空间；不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

```java
    public ListNode reverseKGroup1(ListNode head, int k) {
        ListNode dummy = new ListNode(0);
        // 1->2->3->4->5  k=2
        // 1.声明一个哑节点dummy
        dummy.next = head;
        // 2.pre和end指向同一个前驱节点上
        ListNode pre = dummy, end = dummy;
        while (end.next != null) {
            // 3.获取翻转链表片段 start -> ... -> end
            for (int i = 0; i < k && end != null; i++) {
                end = end.next;
            }
            if (end == null) break;
            ListNode next = end.next;
            // 孤立需翻转链表片段
            end.next = null;
            ListNode start = pre.next;
            // 4.翻转链表 dummy -> end -> ... -> start
            pre.next = reverse(pre);
            // 5.连接翻转后链表，pre和end指向同一个前驱节点上
            start.next = next;
            pre = start;
            end = pre;
        }
        return dummy.next;
    }

    public ListNode reverse(ListNode head) {
        ListNode prev = null;
        while (head != null) {
            ListNode next = head.next;
            head.next = prev;
            prev = head;
            head = next;
        }
        return prev;
    }
```

