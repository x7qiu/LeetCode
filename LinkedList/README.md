# Linked List

## Intro

Most linked list algorithms are logially straightforward. The main difficult lies in that fact that when working on a node in the list, often times you also **need to have reference to the node just before it or after it**. Take for example a simple case of deleting the node `B` from list `A->B->C`. After you identify where `B` is, you still need to keep a reference of `A` in order to write `A->next = B->next`.



This brings code complexity, as you need to keep track of more variables and treat special cases carefully. The head node obviously has no previous node and the last `NULL` node has nothing after it. I usually approach a problem in the following steps:



1. **Design the inside of `while` loop**

   In this step we ignore edge cases and focus on the body of the list. A `cur` node, initialized as the `head` node before entering the loop, will always be the current node that we are working on.

2. **Decide the break condition for the loop**

   Once we have the inside of the loop from above, we break out when one of the referenced node reaches a `NULL` state. 

   This will usually be the `cur` node, but exceptions exsist. For example, if we referenced`cur->next->value` or `cur->next->next` in the loop, we must break out of the loop when `cur->next` is `NULL`.

3. **Handle edge cases**

   The `head` node often needs to be handled seperately because there is no node before it.

## When `pre` node is needed: Dummy node to the rescue

Sometimes you need to keep reference to the node immediately before the `cur` node you are working on, such as when removing all elements of a particular value from the list. The while loop would look like

```c
while (cur){
  if(cur->val == 666){
    pre->next = cur->next;		// deleting cur node from the list
    cur = cur->next;
  }
  else{
    pre = pre->next;
    cur = cur->next;
  }
}
```

However, what if the head node itslef needs to be deleted? Since the head node has no `pre` node before it, we'd have to either adjust the loop logic (which make the code less intuitive) or handle it seperately (which makes the code longer). 

It's a common trick to construct a dummy node to handle such edge cases. The idea is to artificially create a node before the head node so that the loop logic stays consistent, and returns whatever is after the dummy node when we are done. 

#### 203. Remove Linked List Elements

**Input**: 1->2->6->3->4->5->6, val = 6   

**Output**: 1->2->3->4->5

```c
struct ListNode* removeElements(struct ListNode* head, int val) {
    struct ListNode dummy = {-1, head};
    struct ListNode* pre = &dummy;
    struct ListNode* cur = head;
    
    while(cur){
        if(cur->val == val){
          	pre->next = cur->next;
        		cur = cur->next;
        }
        else{
          	pre = cur;
          	cur = cur->next;
        }
    }
    return dummy.next;
}
```

## When `after` node is needed: Reverse, swap the list

Sometimes you need to keep reference to the node after the `cur` node you are working on, such as when reversing the list. This is because you lose what comes after the current node as part of the reversing operation.

I prefer to initialize `after` node in the while loop so my loop condition is short.

#### 206. Reverse Linked List

**Input**: 1->2->3->4->5->NULL

**Output**: 5->4->3->2->1->NULL

```c
struct ListNode* reverseList(struct ListNode* head) {
    struct ListNode* pre = NULL;
    struct ListNode* cur = head;
   
    while (cur){
        struct ListNode* after = cur->next;	// for later reference
        cur->next = pre;	// reverse cur and pre in each iteration
      
        prev = cur;
        cur = after;
    }
    return pre;
}
```

I often find that for such problems, the recursive solution is a bit easier to remember. 

```c
struct ListNode* reverseList(struct ListNode* head) {
    if (head == NULL || head->next == NULL){
        return head;
    }
    
    struct ListNode* tail = reverseList(head->next);
  
  	// suppose the original list is 1->2->3->4->5->NULL
  	// after recursion, tail is 5->4->3->2->NULL, head is 1->2
    head->next->next = head;
    head->next = NULL;
    return tail;
}
```

#### 24. Swap Node in Pairs

**Input**: 1->2->3->4

**Output**: 2->1->4->3

```c
struct ListNode* swapPairs(struct ListNode* head){
    struct ListNode dummy = {-1, head};
    struct ListNode* pre = &dummy;
    struct ListNode* cur = head;
    
    while(cur && cur->next){
        struct ListNode* after = cur->next;
        cur->next = after->next;
        pre->next = after;
        after->next = cur;
        
        pre = pre->next->next;
        cur = pre->next;
        
    }
    return dummy.next;
}
```

```c
struct ListNode* swapPairs(struct ListNode* head) {
    if (head==NULL || head->next == NULL)
        return head;
    
    struct ListNode* rest = swapPairs(head->next->next);
    struct ListNode* newhead = head->next;
    newhead->next = head;
    head->next = rest;
    return newhead;
}
```

## Two Pointers

#### 141. Linked List Cycle

Given a linked list, determine if it has a cycle in it.

```c
bool hasCycle(struct ListNode *head) {
    struct ListNode* slow, *fast;
    slow = head, fast = head;

    while(fast && fast->next){
        slow = slow->next;
        fast = fast->next->next;

        if (slow == fast)
            return true;
    }
    return false;  
}
```

Suppose a cycle exists, then by the time the slow pointer is in it, the fast pointer will already be somewhere inside. From that point, the fast pointer will be catching up to the slow one in the cycle. Since the fast pointer is running twice as fast, it is guarantueed to be one step closer each loop, eventually meeting the slow pointer.

## Other Solved Problems

#### 83. Remove Duplicates from Sorted List

Given a sorted linked list, remove all duplicates such that each element appears only once.

**Input**: 1->1->2->3->3

**Output**: 1->2->3

```c
ListNode* deleteDuplicates(ListNode* head) {
    ListNode *cur = head;
    while (cur && cur->next) {
        if (cur->val == cur->next->val)
            cur->next = cur->next->next;

        cur = cur->next;
    }
    return head;
}
```

```c
struct ListNode* deleteDuplicates(struct ListNode* head) {
    if (head == NULL || head->next == NULL)
        return head;
    
    struct ListNode* rest = deleteDuplicates(head->next);
    if (head->val == rest->val){
        head->next = rest->next;
    }
    return head;
}
```

#### 82. Remove Duplicates From Sorted List II

Given a sorted linked list, remove all nodes that have duplicate elements.

**Input**: 1->2->3->3->4->4->5

**Output**: 1->2->5

```c
struct ListNode* deleteDuplicates(struct ListNode* head){
    struct ListNode dummy = {-1, head};
    struct ListNode* pre = &dummy;
    struct ListNode* cur = head;
    
    while(cur && cur->next){
        if (cur->val == cur->next->val){
            while(cur->next && cur->next->val == cur->val){
                cur = cur->next;	# find the last node with duplicate value
            }
            pre->next = cur->next;
          	cur = cur->next;
        }
        else{
            pre = cur;
            cur = cur->next;
        }
    }
    return dummy.next;
}
```

#### 142. Linked List Cycle II

Given a linked list, return the node where the cycle begins. If there is no cycle, return null.

![detect-loop-start](./images/detect-loop-start.png)

Suppose we have run the tortoise and hare algorithm and they meet at the meeting point as shown in the image. Note that the tortoise is guranteed to be caught up by the hare within one cycle after it enters the cycle. But the hare could have travelled the cycle `m` times before the tortoise enters the cycle. Then we have:

- Distance run by tortoise: x+y
- Distance run by hare: x+m(y+z)+y

Since the hare has been running at twice the speed of the tortoise, we have:
$$
2(x+y)=x+m(y+z)+y
$$
Sovling for x gives:
$$
x=(m−1)(y+z)+z
$$
What this tells us is that in another scenario, if we have two pinters, one starting at the head of the list and the other starting at somewhere in the cycle and running at the same speed. Then by the time the first pointer enters the cycle (aka travelled a distance of x), the second pointer will have moved forward a distance of z relative to its starting position.

Now it should be clear why the following algorithm works:

1. Run the tortoise and hare algorithm. Now both slow and fast pointer are at the meeting point as showin in the image.
2. Move the slow pointer to the beginning of the list again. Set the fast pointer to be running at the same speed as the slow pointer.
3. Move both pointer in parallel until they meet. The new meeting point would be at the start of the cycle.