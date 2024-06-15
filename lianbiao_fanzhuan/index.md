# 链表反转


## 链表反转
[链表反转](https://leetcode.cn/problems/reverse-linked-list/description/)
### 方法一：迭代
在遍历链表时，将当前节点的 next 指针改为指向前一个节点。由于节点没有引用其前一个节点，因此必须事先存储其前一个节点。在更改引用之前，还需要存储后一个节点。最后返回新的头引用。
```golang
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    curr := head
    for curr != nil {
        next := curr.Next
        curr.Next = prev
        prev = curr
        curr = next
    }
    return prev
}
```
复杂度分析

时间复杂度：O(n)，其中n是链表的长度。需要遍历链表一次。

空间复杂度：O(1)。
### 方法二：递归法

```golang
func reverseList(head *ListNode) *ListNode {
  if head == nil || head.Next == nil {
    return head
  }
  newHead := reverseList(head.Next)
  head.Next.Next = head
  head.Next = nil
  return newHead
}
```
复杂度分析

时间复杂度：O(n)，其中n 是链表的长度。需要对链表的每个节点进行反转操作。

空间复杂度：O(n)，其中n 是链表的长度。空间复杂度主要取决于递归调用的栈空间，最多为n层。
