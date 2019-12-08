---
title: LeetCode-Easy
date: 2019-12-08 13:55:09
tags:
---
## Leetcode Easy

### String

**38. Count and Say** 

The count-and-say sequence is the sequence of integers with the first five terms as following:

```
1. 1
2. 11
3. 21
4. 1211
5. 111221
```

`1` is read off as `"one 1"` or `11`.
`11` is read off as` "two 1s"` or `21`.
`21` is read off as `"one 2 , then one 1"` or `1211`.

Given an integer n where 1 ≤ n ≤ 30, generate the nth term of the count-and-say sequence.

```c++
for (int i = 0, j; i < s.length(); i = j) {
    for (j = i; j < s.length() && s[i] == s[j]; ++j);
}
/* j goes first. When s[i] != s[j] or j >= s.length(), update i = j, then goes again */
```



**237. Delete Node in a Linked List**

just give you the node which should be deleted, no list head, given node is not the last on in the list

```c++
void deleteNode(ListNode* node) {
    node->val = node->next->val;
    node->next = node->next->next;   
}
```



**19. Remove Nth Node From End of List**

use two pointer, one is faster than the other N nodes. When the faster one comes to the end, the slower one is the Nth node from end of list;

```c++
ListNode* removeNthFromEnd(ListNode* head, int n) {
    if (head == NULL) {
        return head;
    }
    ListNode *fast = head, *slow = head, *pre = head;
    for (int i = 0; i < n; ++i) {
        fast = fast->next;
    }
    while (fast) {
        pre = slow;
        fast = fast->next;
        slow = slow->next;
    }
    if (slow == head) {
        return head->next;
    }
    pre->next = slow->next;
    return head;
}
```



**206. Reverse List**

```c++
//Recursive
ListNode* reverseList(ListNode* head) {
    if (head == NULL || head->next == NULL) {
        return head;
    }
    ListNode *p = reverseList(head->next);
    head->next->next = head;
    head->next = NULL;
    return p;
}
```

```c++
//Iteration
ListNode* reverseList(ListNode* head) {
    if (head == NULL || head->next == NULL) {
        return head;
    }
    ListNode *pre = head, *now = pre->next;
    while (now) {
        ListNode *tmp = now->next;
        now->next = pre;
        pre = now;
        now = tmp;
    }
    head->next = NULL;
    return now;
}
```



**234. Palindrome Linked List**

double pointer, reverse first half

```c++
bool isPalindrome(ListNode* head) {
    if (head == NULL || head->next == NULL) {
        return true;
    }
    ListNode *fast = head, *slow = head, *pre = head;
    while (fast && fast->next) {
        pre = slow;
        fast = fast->next->next;
        slow = slow->next;
    }
    if (fast && !fast->next) {
        slow = slow->next;
    }
    pre->next = NULL;
    reverseList(head);
    while (slow) {
        if (head->val != slow->val) {
            return false;
        }
        slow = slow->next;
        head = head->next;
    }
    return true;
}
```

