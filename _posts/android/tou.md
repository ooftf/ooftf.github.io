
## 反转链表
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode tail = null;
        ListNode curr = head;
        while (curr != null) {
            ListNode next = curr.next;
            curr.next = tail;
            tail = curr;
            curr = next;
        }
        return tail;
    }
}

class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode newHead = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return newHead;
    }
}
```

