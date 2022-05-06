# 单调栈

leetcode 84题

给定n个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为1。

求在该柱状图中，能够勾勒出来的矩形的最大面积。



​		要求最大面积，不管最后的面积如何，都肯定包括一个完整的柱子。所以我们找最大面积，可以将所有柱子作为那个完整的柱子，也就是最大面积的高度被我们确定。然后向两边扩展，然后比较所有这些扩展的面积大小。选择柱子 j 作为最高的完整柱子，如果向左扩展，那么要求 i < j ，同时heights[i] > heights[j]。然后是边界如何重复利用的问题。 x < y < z ，当我们知道 y 的边界是 x 后，能否能过比较 y 和 z 来判断 z 的边界。对于任意的 i ， x < i < y ，可以说 heights[i] > heights[y] ，如果 heights[y] > heights[z] ，所有的 heights[i] > heights[z] ，所以 z 的边界可以从 x 开始推。这种数据进出模式类似栈。所以可以尝试用栈试试。

​		例子 [6,7,5,2,4,5,9,3] ：

 6 进栈，栈开始为空，无出栈，栈长为1，6的边界height数组第一个数。

7 进栈， 7 > 6 ，无出栈，栈长为2，7的边界为height数组的第二个数。

5 进栈，5 比 6、7小，所以6 、7 出栈，栈长为1，5的边界为height数组的第一个数。

2 进栈，2 < 5，所以5出栈，栈长为1，2的边界为height数组的第一个数。

依次类推。由于数组索引第一个为0，所以可以将所以数字减一。

同理可以得到柱子的右边界。

使用一个栈和两个数组，可以保存数组中任意元素的左右边界。然后比较求值。

```go
func largestRectangleArea(heights []int) int {
    n := len(heights)
    left, right := make([]int, n), make([]int, n)
    for i := 0; i < n; i++ {
        right[i] = n
    }
    mono_stack := []int{}
    for i := 0; i < n; i++ {
        for len(mono_stack) > 0 && heights[mono_stack[len(mono_stack)-1]] >= heights[i] {
            right[mono_stack[len(mono_stack)-1]] = i
            mono_stack = mono_stack[:len(mono_stack)-1]
        }
        if len(mono_stack) == 0 {
            left[i] = -1
        } else {
            left[i] = mono_stack[len(mono_stack)-1]
        }
        mono_stack = append(mono_stack, i)
    }
    ans := 0
    for i := 0; i < n; i++ {
        ans = max(ans, (right[i] - left[i] - 1) * heights[i])
    }
    return ans
}


func max(x, y int) int {
    if x > y {
        return x
    }
    return y
}
```



