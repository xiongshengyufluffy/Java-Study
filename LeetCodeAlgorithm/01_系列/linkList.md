https://zhuanlan.zhihu.com/p/85122573

## 链表排序
### 147. 对链表进行插入排序
```
class Solution:
    def insertionSortList(self, head: ListNode) -> ListNode:
        if not head or not head.next:
            return head # 链表中没有节点或只有一个节点
        dummy = ListNode(-float("inf"))
        dummy.next = head
        cur_prev = dummy
        cur = head
        
        while cur:
            cur_next = cur.next # 当前节点的下一个节点
            # 1 [2]p [3]cur 3 4 
            if cur.val >= cur_prev.val:
                cur = cur.next
                cur_prev = cur_prev.next
            # 1 3 4 [5]p [2]cur 6
            else:
                # 插入排序
                node = dummy
                while node.next:
                    if node.next.val <= cur.val:
                        node = node.next
                    else:
                        break
                # node.val <= cur.val < node.next.val
                temp_node_next = node.next
                node.next = cur
                cur.next = temp_node_next
                
                cur_prev.next = cur_next
                # 完成插入之后,当前节点的前驱位置不发生改变
                cur = cur_next
                
        
        return dummy.next
```

## 1220复习
### 25. K 个一组翻转链表
- Java
```
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode result = recurse(head, k);
        // printLink(result);
        return result;
    }
    public ListNode recurse(ListNode head, int k){
        int length = k; // 先计算出链表的总长度
        ListNode end = head;
        // ListNode preEnd = head; // end 的前驱
        while(end != null){
            end = end.next;
            // preEnd = preEnd.next;
            // if(preEnd.next == null) break;
            length --;
            if(length == 0) break;
        }
        if (length > 0) return head;
        // ListNode preEnd.next = null;
        ListNode start = head;
        ListNode nextHead = recurse(end, k);
        ListNode curHead = reverseList(start, end), curTail = start;
        curTail.next = nextHead;
        return curHead;
    }

    public ListNode reverseList(ListNode startNode, ListNode endNode) {
        int length = 1; 
        ListNode prev = null, cur = startNode;
        while(cur != endNode){
            ListNode next = cur.next; //|prev|->|cur|->|next| |prev|<-|cur|<-|next|
            cur.next = prev;
            prev = cur;
            cur = next;
            length ++;
        }
        return prev;
    }

    // public void printLink(ListNode node){
    //     while (node != null){
    //         System.out.print(node.val + "-->");
    //         node = node.next;
    //     }
    //     System.out.println(" ");
    // }
}
```