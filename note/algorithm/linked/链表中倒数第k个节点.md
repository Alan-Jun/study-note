输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。

例如，一个链表有 6 个节点，从头节点开始，它们的值依次是 1、2、3、4、5、6。这个链表的倒数第 3 个节点是值为 4 的节点。

 

示例：

给定一个链表: 1->2->3->4->5, 和 k = 2.

返回链表 4->5.

> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

```java
public ListNode getKthFromEnd(ListNode head, int k) {
    ListNode leftNode = head;
    ListNode kNode = head;
    // kNode 先移动k个数，然后让 leftNode 和 kNode 同时后移直到 kNode移动到尾部这时候leftNode就是倒数第k个节点
    int num = 1; // kNode 初始阶段是占用了一个节点的，所以 num 从 1 开始
    while (num < k) {
        kNode = kNode.next;
        num++;
    }
    while (kNode.next!=null){
        kNode = kNode.next;
        leftNode = leftNode.next;
    }
    return leftNode;
}
```

更好的方式是我们在设计链表的时候记录链表的总长，这样直接就可以计算得出需要向后移动多少次，如果设计的是双向链表还可以根据计算出来的路径采取从head向后，或从tail向前查询的最优解，甚至可以直接使用数组来作为链表的容器，这样可以让这样的查询变成O(1) 的查询