# Linked List

## Intro

Most linked list algorithms are logially straightforward. The main difficult lies in that fact that when working on a node in the list, often times you also **need to have reference to the node just before it or after it**. Take for example a simple case of deleting the node `B` from list `A->B->C`. After you identify where `B` is, you still need to keep a reference of `A` in order to write `A->next = B->next`.



This brings code complexity, as you need to keep track of more variables and treat special cases carefully. The head node obviously has no previous node and the last `NULL` node has nothing after it.  Recursive solutions often exist and are easier to reason about, but bring up the space complexity to O(n). For an iterative solution, I usually formulate it with the following steps:



1. **Design the inside of `while` loop**

   In this step we ignore edge cases and focus on the body of the list. A `cur` node, initialized as the `head` node before entering the loop, will always be the current node that we are looking at.

2. **Decide the break condition for the loop**

   Once we have the inside of the loop from above, we break out when one of the referenced node reaches  `NULL`. 

   This will usually be the `cur` node, but exceptions exsist. For example, if we referenced`cur->next->value` or `cur->next->next` in the loop, we must break out of the loop when `cur->next` is `NULL`.

3. **Handle edge cases**

   The `head` node needs to be handled seperately if we referenced a `pre` node inside the loop, because `head` has no previous node. Similary, if we break out of the loop before `cur` reaches `NULL`, the last nodes need to be handled on its own. 

## When `pre` node is needed: Dummy node to the rescue

Sometimes you need to keep reference to the node immediately before the `cur` node you are working on, such as when removing all elements of a particular value from the list. The while loop would look like

```c
while (cur){
    if(cur->val == 666){
        // delete current node
        pre->next = cur->next;		
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

* **Input**: 1->2->6->3->4->5->6->NULL, val = 6   

* **Output**: 1->2->3->4->5->NULL

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
            pre = pre->next;
            cur = cur->next;
        }
    }
    return dummy.next;
}
```

## When `after` node is needed: Reverse, swap the list

Sometimes you need to keep reference to the node after the `cur` node if your code reassigns `cur->next`, such as reversing the list.

I prefer to initialize `after` node in the loop body so my break condition is short.

#### 206. Reverse Linked List

* **Input**: 1->2->3->4->5->NULL

* **Output**: 5->4->3->2->1->NULL

```c
struct ListNode* reverseList(struct ListNode* head) {
    struct ListNode* pre = NULL;
    struct ListNode* cur = head;
   
    while (cur){
        struct ListNode* after = cur->next;	// for later reference
        cur->next = pre;			            // reverse cur and pre
      
        prev = cur;
        cur = after;
    }
    return pre;
}
```

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

* **Input**: 1->2->3->4->NULL

* **Output**: 2->1->4->3->NULL

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

Since you cannot go back in a singly linked list, sometimes it's useful to keep reference of two or more pointers to make certain tasks easier.

#### 876. Middle of the Linked List

Return the middle node of a linked list. If there are two middle nodes, return the second one.

```c
struct ListNode* middleNode(struct ListNode* head) {
    if (head == NULL || head->next == NULL)
        return head;
    struct ListNode* slow, *fast;
    slow = head, fast = head;

    while(fast && fast->next){
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
```



## Solved Problems

#### 83. Remove Duplicates from Sorted List

Given a sorted linked list, remove all duplicates such that each element appears only once.

* **Input**: 1->1->1->2->3->3->NULL

* **Output**: 1->2->3->NULL

We never need to remvoe the current node so there is no need for dummy node. For current node, compare its value to the next node and **remove the next node** if they are the same. Only move the current node forward if the next code has a different value.

```c
struct ListNode* deleteDuplicates(struct ListNode* head) {
    struct ListNode *cur = head;
    while (cur && cur->next) {
        if (cur->val == cur->next->val){
       	//remove cur->next; do not move cur forward
        cur->next = cur->next->next;
        }
        else{
            cur = cur->next;
        }
    }
    return head;
}
```

Since we broke out of the loop when `cur->next` is `NULL`, we need to examine if the last node needs to be handled seperately. 

#### 82. Remove Duplicates From Sorted List II

Given a sorted linked list, remove all nodes that have duplicate elements.

* **Input**: 1->2->3->3->4->4->5->NULL

* **Output**: 1->2->5->NULL

```c
struct ListNode* deleteDuplicates(struct ListNode* head){
    struct ListNode dummy = {-1, head};
    struct ListNode* pre = &dummy;
    struct ListNode* cur = head;
    
    while(cur && cur->next){
        if (cur->val == cur->next->val){
          // find the last node with duplicate value
            while(cur->next && cur->next->val == cur->val){
                cur = cur->next;	
            }
          // break out when cur needs to be deleted i.e. cur->next has a distinct value
            pre->next = cur->next;
          	cur = cur->next;
        }
        else{
            pre = pre->next;
            cur = cur->next;
        }
    }
    return dummy.next;
}
```

Instead of thinking about when to remove nodes from the original list, we can also contruct a new list and think about when to add unique nodes to it.

```c
struct ListNode* deleteDuplicates(struct ListNode* head){
    if (!head) return NULL;

    // needs a dummy node for 'pre'; ensures it has different value from head
    struct ListNode dummy = {head->val -1, head};   
    struct ListNode* pre = &dummy;
    struct ListNode* cur = head;
    
    // add distinct nodes to this new list
    struct ListNode* newList = &dummy; 
    
    while(cur){
        if (cur->val != pre->val && (cur->next == NULL || cur->val != cur->next->val)){
            // this is a unique node. Add to the new list
            newList->next = cur;
            newList = newList->next;
        }
        pre = pre->next;
        cur = cur->next;
    }
    
    // end of new list
    newList->next = NULL;
    
    return dummy.next;
}
```

Note that in the loop condition is simplay `while(cur)` even though we are referencing `cur->next->val` in the loop body. That and the inclusion of `cur->next == NULL` in the `if` condition is necessary to make the code concise because it takes care of the last node (before NULL). 

If the loop condition is `while(cur && cur->next)` like we normally do, then the loop break at the last node and we will have to handle it seperately.

#### Todo

WHY is it that the last node needs special handling here, but not previous examples?

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

#### 142. Linked List Cycle II

Given a linked list, return the node where the cycle begins. If there is no cycle, return null.

![detect-loop-start](./images/detect-loop-start.png)

Suppose we have run the tortoise and hare algorithm and they meet at the meeting point as shown in the image. Note that the tortoise is guranteed to be caught up by the hare within one cycle after it enters the cycle. But the hare could have travelled the cycle `m` times before the tortoise enters the cycle. Then we have:

- Distance run by tortoise: `x+y`
- Distance run by hare: `x+m(y+z)+y`

Since the hare has been running at twice the speed of the tortoise, we have:
$$
2(x+y)=x+m(y+z)+y
$$
Sovling for x gives:
$$
x=(m???1)(y+z)+z
$$

- LHS is the distance from head to the start of the cycle.
- RHS is the distance from meeting point to the start of the cycle, plus `m-1` cycles.

Now it should be clear why the following algorithm works:

1. Run the tortoise and hare algorithm. Now both slow and fast pointer are at the meeting point.
2. Move the slow pointer to the head of the list again. Set the fast pointer to be running at the same speed as the slow pointer.
3. Move both pointer in parallel until they meet. They meet at the start of the cycle.

```c
struct ListNode *detectCycle(struct ListNode *head) {
    struct ListNode* slow, *fast;
    slow = head, fast = head;   
 
    while(fast && fast->next){
        slow = slow->next;
        fast = fast->next->next;
        
        if (slow == fast){
            slow = head;
            while(slow != fast){
                slow = slow->next;
                fast = fast->next;
            }
            return slow;
        }
    }
    return NULL;
}
```

#### 234. Palindrome Linked List

Given the head of a linked list, return true if it is a palindrome.

* **Input**: 1->2->2->1->NULL

* **Output**: true

```python
def isPalindrome(self, head):
    vals = []
    while head:
        vals.append(head.val)
        head = head.next
    return vals == vals[::-1]
```

Sometimes this approach is forbidden for some made-up reasons, or O(1) extra space is required. In such cases we can combine techniques we've seen so far. e.g. for this problem we can identify the middle node, reverse either half of the list and then compare if they are the same. However, the reverse operation is destructive to the original list.

There is also a recursive solution to this problem, which takes O(n) extra space, does not destory the original list, and is quite pleasing to the eye. It's unlikely I will ever come up with it on my own during an interview, but finding such cleaver use of recursion is what makes the fked up interview process less miserable.

```c
bool isPalindrome(struct ListNode* head){
    struct ListNode* front = head;
    return helper(&front, head);
}

bool helper(struct ListNode** front, struct ListNode* back){
    if (back == NULL)   return true;
    
    // let back pointer travels to the end of the list through recursion;
    bool equal_so_far = helper(front, back->next);
    
    // *front and back should have the same value
    bool value_equal = (back->val == (*front)->val);
    
    // when the functions returns, back is gradually moved towards head of the list 
    // so we need to move front accordingly to compare their values
    // front must be global or passed in by refernece so that the change is not lost
    *front = (*front)->next;
    return equal_so_far && value_equal;
}
```

#### 160. Intersection oof Two Linked Lists

Write a program to find the node at which the intersection of two singly linked lists begins.

- If the two linked lists have no intersection at all, return null.
- You may assume there are no cycles anywhere in the entire linked structure.

1. Traverse the lists and compute their lengths, called it `lenA` and `lenB`
2. Move the head of the longer list `abs(lenA-lenB)` nodes ahead, so that now headA and headB are sitting at the same distance from tail
3. Traverse headA and headB in parallel and compare the address of each node along the way. If they are equal, it is the intersection point

```python
def getIntersectionNode(headA, headB):
        curA,curB = headA,headB
        lenA,lenB = 0,0
        while curA is not None:
            lenA += 1
            curA = curA.next
        while curB is not None:
            lenB += 1
            curB = curB.next
        curA,curB = headA,headB
        if lenA > lenB:
            for i in range(lenA-lenB):
                curA = curA.next
        elif lenB > lenA:
            for i in range(lenB-lenA):
                curB = curB.next
        while curA and id(curA) != id(curB):
            curA = curA.next
            curB = curB.next
        return curA
```



