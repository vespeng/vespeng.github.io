# 算法实现示例集：链表


链表是数据结构中的重要基础，它通过指针连接零散的内存节点，体现了动态数据结构的核心思想。掌握链表的核心操作都是程序员的关键能力。

本文不追求冗长的理论推导，而是聚焦于提供清晰、可运行的核心代码实现。
<!--more-->

## 反转链表

题目链接：[https://leetcode.cn/problems/reverse-linked-list/](https://leetcode.cn/problems/reverse-linked-list/)

基本思路：使用迭代法，维护三个指针：prev（前驱）、curr（当前）和 next（后继）。在遍历过程中，逐个改变节点的指向。

图解：

> 初始: 1 -> 2 -> 3 -> 4 -> 5 -> nil
>
> 第一步: nil <- 1 2 -> 3 -> 4 -> 5 (prev=nil, curr=1, next=2， 将curr.Next指向prev)
> 
> 第二步: nil <- 1 <- 2 3 -> 4 -> 5 (指针移动，重复上述过程)
>
> 完成: nil <- 1 <- 2 <- 3 <- 4 <- 5 (新的头节点是5)

```go {data-open=true}
type ListNode struct {
    Val int
    Next *ListNode
}

func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    curr := head
    for curr != nil {
        nextTemp := curr.Next // 保存下一个节点
        curr.Next = prev      // 反转当前节点的指针
        prev = curr           // prev 前移
        curr = nextTemp       // curr 后移
    }
    return prev // 返回新的头节点
}
```

## 环形链表判断

题目链接：[https://leetcode.cn/problems/linked-list-cycle/](https://leetcode.cn/problems/linked-list-cycle/)

基本思路：使用快慢指针（Floyd判圈算法）。快指针每次走两步，慢指针每次走一步。如果链表有环，快慢指针最终会相遇。

示例：

> 输入: head = [3,2,0,-4], pos = 1 (形成环)
> 
> 快慢指针移动:
> 
> 步1: slow=3, fast=2
> 
> 步2: slow=2, fast=0
> 
> 步3: slow=0, fast=2
> 
> 步4: slow=-4, fast=-4 (相遇，说明有环)

```go {data-open=true}
func hasCycle(head *ListNode) bool {
    if head == nil {
        return false
    }
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
        if slow == fast {
            return true
        }
    }
    return false
}
```

## 合并两个有序链表

题目链接：[https://leetcode.cn/problems/merge-two-sorted-lists/](https://leetcode.cn/problems/merge-two-sorted-lists/)

基本思路：使用递归或迭代。迭代法创建一个哑节点（dummy node）作为新链表的起始点，比较两个链表的节点值，将较小的节点依次连接到新链表上。

示例：

> 输入: l1 = 1 -> 3 -> 5, l2 = 2 -> 4 -> 6
> 
> 比较: 1 < 2 -> 新链表接1
> 
> 比较: 3 > 2 -> 新链表接2
> 
> 最终结果: 1 -> 2 -> 3 -> 4 -> 5 -> 6

```go {data-open=true}
func mergeTwoLists(list1 *ListNode, list2 *ListNode) *ListNode {
    // 哑节点，简化边界处理
	dummy := &ListNode{}
    tail := dummy

    for list1 != nil && list2 != nil {
        if list1.Val <= list2.Val {
            tail.Next = list1
            list1 = list1.Next
        } else {
            tail.Next = list2
            list2 = list2.Next
        }
        tail = tail.Next
    }
    // 将剩余非空链表直接接上
    if list1 != nil {
        tail.Next = list1
    } else {
        tail.Next = list2
    }
    return dummy.Next
}
```

## 删除链表的倒数第 N 个结点

题目链接：[https://leetcode.cn/problems/remove-nth-node-from-end-of-list/](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

基本思路：使用双指针，让快指针先走 N 步，然后快慢指针同步向后移动。当快指针到达链表末尾时，慢指针正好指向待删除节点的前一个节点，执行删除操作。

图解：

> 链表: 1 -> 2 -> 3 -> 4 -> 5, n = 2
> 
> 初始: fast = 1, slow = 1 (dummy)
> 
> fast 先走2步: fast = 3, slow = 1
> 
> 同步移动: fast到5(nil)时，slow到3 (要删除4，需让3.Next指向5)

```go {data-open=true}
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    // 哑节点，防止删除头节点时出错
    dummy := &ListNode{Next: head}
    fast, slow := dummy, dummy
	
    // 快指针先走 n 步
    for i := 0; i < n; i++ {
        fast = fast.Next
    }
    // 快慢指针同步移动，直到快指针到达末尾
    for fast.Next != nil {
        fast = fast.Next
        slow = slow.Next
    }
    // 删除慢指针后面的节点
    slow.Next = slow.Next.Next
    return dummy.Next
}
```

## 相交链表判断

题目链接：[https://leetcode.cn/problems/intersection-of-two-linked-lists/](https://leetcode.cn/problems/intersection-of-two-linked-lists/)

基本思路：使用双指针技巧。指针A遍历链表A后遍历链表B，指针B遍历链表B后遍历链表A。如果两个链表相交，指针A和指针B会在相交点相遇（因为走的总长度相同）。

示例：

> 链表A: a1 -> a2 -> c1 -> c2
> 
> 链表B: b1 -> b2 -> b3 -> c1 -> c2
> 
> 指针A路径: a1 a2 c1 c2 b1 b2 b3 c1
> 
> 指针B路径: b1 b2 b3 c1 c2 a1 a2 c1
> 
> 在c1处相遇

```go {data-open=true}
func getIntersectionNode(headA, headB *ListNode) *ListNode {
    if headA == nil || headB == nil {
        return nil
    }
    pa, pb := headA, headB
    for pa != pb {
        if pa == nil {
            pa = headB
        } else {
            pa = pa.Next
        }
        if pb == nil {
            pb = headA
        } else {
            pb = pb.Next
        }
    }
    // 要么相遇于交点，要么同时为nil（不相交）
    return pa
}
```


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.com/posts/algorithm_implementation_example_linked_list/  

