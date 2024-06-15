# 环形链表


## 环形链表
[环形链表](https://leetcode.cn/problems/linked-list-cycle/)
### 方法一：哈希表
使用哈希表来存储所有已经访问过的节点。每次我们到达一个节点，如果该节点已经存在于哈希表中，则说明该链表是环形链表，否则就将该节点加入哈希表中。重复这一过程，直到我们遍历完整个链表即可。
```golang
func hasCycle(head *ListNode) bool {
	seen := map[*ListNode]struct{}{}
    for head != nil {
		if _, ok = seen[head]; ok {
			return true
        }
		seen[head] = struct{}{}
        head  = head.Next
    }   
	return false
}
```
复杂度分析

时间复杂度：O(n)，其中n 是链表的长度。最坏情况下我们需要遍历每个节点一次。

空间复杂度：O(n)，其中n 是链表的长度。主要为哈希表的开销，最坏情况下我们需要将每个节点插入到哈希表中一次。

### 方法二：快慢指针
具体地，我们定义两个指针，一快一慢。慢指针每次只移动一步，而快指针每次移动两步。初始时，慢指针在位置 head，而快指针在位置 head.next。这样一来，如果在移动的过程中，快指针反过来追上慢指针，就说明该链表为环形链表。否则快指针将到达链表尾部，该链表不为环形链表。
```golang
func hasCycle(head *ListNode) bool {
    if head == nil || head.Next == nil {
        return false
    }
    slow, fast := head, head.Next
	for slow != fast {
		if fast == nil || fast.Next == nil {
			return false
        }     
		slow = slow.Next
		fast = fast.Next.Next
    }   
	return true
}
```
复杂度分析

时间复杂度：O(n)，其中n 是链表的长度。

当链表中不存在环时，快指针将先于慢指针到达链表尾部，链表中每个节点至多被访问两次。

当链表中存在环时，每一轮移动后，快慢指针的距离将减小一。而初始距离为环的长度，因此至多移动 N 轮。

空间复杂度：O(1)，我们只使用了两个指针的额外空间。

