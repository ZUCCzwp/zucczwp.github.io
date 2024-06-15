# 链表的中间结点


## 链表的中间结点
[链表的中间结点](https://leetcode.cn/problems/middle-of-the-linked-list/)
### 方法一：快慢指针
用两个指针 slow 与 fast 一起遍历链表。slow 一次走一步，fast 一次走两步。那么当 fast 到达链表的末尾时，slow 必然位于中间。
```golang
func middleNode(head *ListNode) *ListNode {
    // 快慢指针，快指针走2步，慢指针走1步
    if head.Next == nil {
        return head
    }
    slow, fast := head, head
    
    for fast != nil && fast.Next != nil {
        fast = fast.Next.Next
        slow = slow.Next
    }
    return slow
}
```
复杂度分析

时间复杂度：O(n)，其中n是链表的长度。需要遍历链表一次。

空间复杂度：O(1)。只需要常数空间存放 slow 和 fast 两个指针。
