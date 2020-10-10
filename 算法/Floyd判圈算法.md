# Floyd判圈算法

## 算法概述

Floyd 判圈算法（Floyd Cycle Detection Algorithm），又称龟兔赛跑算法（Tortoise and Hare Algorithm）。

是一个可以在有限状态机、迭代函数或者链表上判断是否存在环，求出该环的起点与长度的算法。

## 思路

设两个指针`t`和`h`已不同步伐（`t`一步`h`两步）从起点出发，如果存在环，两个指针必有某时刻相遇。

## 解决问题

### 是否存在环

设两个指针`t`和`h`已不同步伐（`t`一步`h`两步）从起点出发：

1. 如果快指针到达了链表尾部，两者都没相遇，则没有环。
2. 如果两个指针相遇，则有环。

### 环的长度

两个指针相遇，设相遇点为`M`，让`h`指针留在`M`点，`t`指针继续前进，每次一步。再次到达`M`点时，前进的步数即为环的长度。

### 环的起点

两个指针相遇，设相遇点为`M`，让`h`指针留在`M`点，`t`指针回到原点`head`，两个指针同时已每次一步的步伐前进，再次相遇时即为环的起点。

#### **证明**

**假设**

相遇时慢指针走过节点数`i`

链表起点到环的起点长度为`m`

环的长度为`n`

相遇的位置与环的起点距离为`k`

**则有**

`i = m + a * n + k`

因为快指针速度为慢指针2倍

`2 * i = m + b * n + k`

`a` `b`分别为慢指针和快指针绕环的圈数。

两式相减得

`i = (b - a) * n`

即**`i`为环的长度的整数倍**

此时将`t`指针放回起点`head`，两指针均已每次一步的速度前进，当`t`指针走到环起点时，前进了`m`步，`h`指针与`head`的距离为`i + m`。又因为**`i`为环的长度的整数倍**，所以`h`指针此时也在环的起点上。

## 代码实现

```java
//leetcode142:Floyd判圈算法
public ListNode floydCycle(ListNode head) {
    if (head == null || head.next == null) return null;
    ListNode slow = head.next;
    ListNode fast = head.next.next;
    while (fast != null && fast.next != null) {
        if (fast == slow) {
            fast = head;
            while(fast != slow) {
                slow = slow.next;
                fast = fast.next;
            }
            return  fast;
        }
        slow = slow.next;
        fast = fast.next.next;
    }
    return null;
}
```

# 参考资料

[Floyd判圈算法（龟兔赛跑算法）](https://blog.csdn.net/xiaoquantouer/article/details/51620657)

[LeetCode 第 142 号问题：环形链表 II](https://github.com/MisterBooo/LeetCodeAnimation/blob/master/0142-Linked-List-Cycle-ii/Article/0142-Linked-List-Cycle-ii.md)