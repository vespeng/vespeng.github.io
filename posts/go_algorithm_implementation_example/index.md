# Go 基础算法实现示例集


在工作与面试准备中，常常需要快速回顾基础算法的核心实现。本文正是一份为此类场景打造的 Go 语言算法速查手册。

本文不追求冗长的理论推导，而是聚焦于提供清晰、可运行的核心代码实现。内容涵盖了链表、字符串处理与排序算法等关键主题，旨在成为一份可以随时查阅、即拿即用的代码参考，助高效巩固基础。

## 排序

### 冒泡排序

基本思路：通过重复遍历要排序的列表，比较相邻的两个元素，如果它们的顺序错误，则交换它们的位置，直到所有元素排序完成。

时间复杂度：\(O(n^2)\)，n 为数组的长度。

图解：

<img src="./images/bubble_sort.gif" alt="冒泡排序" style="width: 55%; height: auto;">

```go
func bubbleSort(arr []int) {
    for i := 0; i < len(arr)-1; i++ {
        for j := 0; j < len(arr)-i-1; j++ {
			// 比较相邻元素，如果前一个元素大于后一个元素，则交换它们
            if arr[j] > arr[j+1] {
				arr[j], arr[j+1] = arr[j+1], arr[j]
            }
        }
    }
}
```

### 选择排序

基本思路：循环遍历列表，每轮从未排序部分选择最小元素放到已排序部分的末尾，直到所有元素排序完成。

时间复杂度：\(O(n^2)\)，n 为数组的长度。

图解：

<img src="./images/selection_sort.gif" alt="选择排序" style="width: 55%; height: auto;">

```go
func selectionSort(arr []int) {
	for i := 0; i < len(arr)-1; i++ {
		minIndex := i
		for j := i + 1; j < len(arr); j++ {
			// 找到最小的元素索引
			if arr[j] < arr[minIndex] {
				// 更新最小元素索引
				minIndex = j
            }
        }
		// 交换元素
		arr[i], arr[minIndex] = arr[minIndex], arr[i]
    }
}
```

### 插入排序

基本思路：将数组分为已排序部分和未排序部分，从未排序部分中取出一个元素，将其插入已排序部分的正确位置，直到所有元素排序完成。

时间复杂度：\(O(n^2)\)，n为数组的长度。

图解：

<img src="./images/insertion_sort.gif" alt="插入排序" style="width: 55%; height: auto;">

```go
func insertionSort(arr []int) {
    for i := 1; i < len(arr); i++ {
        key := arr[i]
        j := i - 1
        // 将大于 key 的元素向后移动
        for j >= 0 && arr[j] > key {
            arr[j+1] = arr[j]
            j--
        }
		// 将 key 插入已排序部分的正确位置
		arr[j+1] = key
	}
}
```

## 链表

### 反转链表

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

```go
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

### 环形链表判断

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

```go
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

### 合并两个有序链表

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

```go
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

### 删除链表的倒数第 N 个结点

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

```go
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

### 相交链表判断

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

```go
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

## 字符串

### 反转字符串

题目链接：[https://leetcode.cn/problems/reverse-string/](https://leetcode.cn/problems/reverse-string/)

基本思路：使用双指针技术，一个指针从字符串开头向中间移动，另一个从末尾向中间移动，交换两个指针所指的字符。

图解：

> 初始: ['h','e','l','l','o']
>
> 第一步: ['o','e','l','l','h'] (交换h和o)
>
> 第二步: ['o','l','l','e','h'] (交换e和l)
>
> 完成: ['o','l','l','e','h']

```go
func reverseString(s []byte) {
    left, right := 0, len(s)-1
    for left < right {
        s[left], s[right] = s[right], s[left]
        left++
        right--
    }
}
```

### 验证回文串

题目链接：[https://leetcode.cn/problems/valid-palindrome/](https://leetcode.cn/problems/valid-palindrome/)

基本思路：使用双指针技术，在忽略非字母数字字符且忽略大小写的情况下，判断字符串正反读是否相同。

示例：

> 输入: "A man, a plan, a canal: Panama"
>
> 处理: "amanaplanacanalpanama"
>
> 结果: true（是回文串）

```go
func isPalindrome(s string) bool {
    s = strings.ToLower(s)
    left, right := 0, len(s)-1
    for left < right {
		// 跳过非字母数字字符
        for left < right && !isAlnum(s[left]) {
            left++
        }
		// 跳过非字母数字字符
        for left < right && !isAlnum(s[right]) {
            right--
        }
		// 判断字符是否相同
        if s[left] != s[right] {
            return false
        }
        left++
        right--
    }
    return true
}

func isAlnum(c byte) bool {
    return (c >= 'a' && c <= 'z') || (c >= '0' && c <= '9')
}
```

### 最长公共前缀

题目链接：[https://leetcode.cn/problems/longest-common-prefix/](https://leetcode.cn/problems/longest-common-prefix/)

基本思路：纵向扫描，以第一个字符串为基准，比较所有字符串的对应位置字符。

示例：

> 输入: ["flower","flow","flight"]
>
> 比较:
>
> 第1列: f,f,f → 相同
>
> 第2列: l,l,l → 相同
>
> 第3列: o,o,i → 不同 → 结果: "fl"

```go
func longestCommonPrefix(strs []string) string {
    if len(strs) == 0 {
        return ""
    }
    for i := 0; i < len(strs[0]); i++ {
        for j := 1; j < len(strs); j++ {
			// 遇到不同字符，返回结果
            if i >= len(strs[j]) || strs[j][i] != strs[0][i] {
                return strs[0][:i]
            }
        }
    }
    return strs[0]
}
```

### 字符串相加

题目链接：[https://leetcode.cn/problems/add-strings/](https://leetcode.cn/problems/add-strings/)

基本思路：模拟竖式加法，从字符串末尾开始逐位相加，处理进位。

示例：

> 输入: "123", "456"
>
> 计算:
>
> 个位: 3+6=9 → 进位0, 当前位9
>
> 十位: 2+5=7 → 进位0, 当前位7
>
> 百位: 1+4=5 → 进位0, 当前位5
>
> 结果: "579"

```go
func addStrings(num1 string, num2 string) string {
    var res []byte
	// 从末尾开始计算
    i, j := len(num1)-1, len(num2)-1
	// 进位
    carry := 0
    
    for i >= 0 || j >= 0 || carry != 0 {
        var x, y int
        if i >= 0 {
            x = int(num1[i] - '0')
            i--
        }
        if j >= 0 {
            y = int(num2[j] - '0')
            j--
        }
        
        sum := x + y + carry
        res = append(res, byte(sum%10+'0'))
        carry = sum / 10
    }
    
    // 反转结果
    for l, r := 0, len(res)-1; l < r; l, r = l+1, r-1 {
        res[l], res[r] = res[r], res[l]
    }
    return string(res)
}
```

### 最长不重复子串

题目链接：[https://leetcode.cn/problems/longest-substring-without-repeating-characters/](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

基本思路：滑动窗口技术，维护窗口和字符位置映射，遇到重复字符时调整窗口起始位置。

示例：

> 输入: "abcabcbb"
>
> 窗口变化:
>
> [a]bcabcbb → [ab]cabcbb → [abc]abcbb
>
> 发现a重复: a[bca]bcbb → 继续滑动...
>
> 最长子串: "abc" (长度3)

```go
func lengthOfLongestSubstring(s string) int {
    maxLen := 0
    left := 0
    charIndex := make(map[byte]int)
    
    for right := 0; right < len(s); right++ {
		// 遇到重复字符，更新窗口起始位置
        if idx, exists := charIndex[s[right]]; exists && idx >= left {
            left = idx + 1
        }
		// 维护字符位置映射
        charIndex[s[right]] = right
        if right-left+1 > maxLen {
            maxLen = right - left + 1
        }
    }
    return maxLen
}
```


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.com/posts/go_algorithm_implementation_example/  

