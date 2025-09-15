# 算法实现示例集：字符串


字符串处理是工作中基础且常用的操作，无论是面试准备、日常开发还是学习巩固，掌握字符串处理的核心算法都是程序员必备技能。

本文提供清晰、可运行的 Golang 核心代码实现，快速掌握字符串处理的核心算法。
<!--more-->

## 反转字符串

题目链接：[https://leetcode.cn/problems/reverse-string/](https://leetcode.cn/problems/reverse-string/)

基本思路：使用双指针技术，一个指针从字符串开头向中间移动，另一个从末尾向中间移动，交换两个指针所指的字符。

图解：

> 初始：['h','e','l','l','o']
> 
> 第一步：['o','e','l','l','h'] (交换h和o)
> 
> 第二步：['o','l','l','e','h'] (交换e和l)
> 
> 完成：['o','l','l','e','h']

```go {data-open=true}
func reverseString(s []byte) {
    left, right := 0, len(s)-1
    for left < right {
        s[left], s[right] = s[right], s[left]
        left++
        right--
    }
}
```

## 验证回文串

题目链接：[https://leetcode.cn/problems/valid-palindrome/](https://leetcode.cn/problems/valid-palindrome/)

基本思路：使用双指针技术，在忽略非字母数字字符且忽略大小写的情况下，判断字符串正反读是否相同。

示例：

> 输入: "A man, a plan, a canal: Panama"
> 
> 处理: "amanaplanacanalpanama"
> 
> 结果: true（是回文串）

```go {data-open=true}
func isPalindrome(s string) bool {
    s = strings.ToLower(s)
    left, right := 0, len(s)-1
    for left < right {
        for left < right && !isAlnum(s[left]) {
            left++
        }
        for left < right && !isAlnum(s[right]) {
            right--
        }
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

## 最长公共前缀

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

```go {data-open=true}
func longestCommonPrefix(strs []string) string {
    if len(strs) == 0 {
        return ""
    }
    for i := 0; i < len(strs[0]); i++ {
        for j := 1; j < len(strs); j++ {
            if i >= len(strs[j]) || strs[j][i] != strs[0][i] {
                return strs[0][:i]
            }
        }
    }
    return strs[0]
}
```

## 字符串相加

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

```go {data-open=true}
func addStrings(num1 string, num2 string) string {
    var res []byte
    i, j := len(num1)-1, len(num2)-1
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

## 最长不重复子串

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

```go {data-open=true}
func lengthOfLongestSubstring(s string) int {
    maxLen := 0
    left := 0
    charIndex := make(map[byte]int)
    
    for right := 0; right < len(s); right++ {
        if idx, exists := charIndex[s[right]]; exists && idx >= left {
            left = idx + 1
        }
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
> URL: https://vespeng.com/posts/algorithm_implementation_example_string/  

