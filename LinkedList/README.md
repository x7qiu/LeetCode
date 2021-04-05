# LinkedList

Most linked list algorithms are logially straightforward. The main difficult lies in that fact that when working on a node in the list, often times you also **need to have reference to the node just before it or after it**. Take for example a simple case of deleting the node `B` from list `A->B->C`. After you identify where `B` is, you still need to keep a reference of `A` in order to write `A->next = B->next`.



This brings code complexity, as you need to keep track of more variables and treat special cases carefully. The head node obviously has no previous node and the last `NULL` node has nothing after it. I usually approach a problem in the following steps:



1. **Design the inside of `while` loop**

   In this step we ignore edge cases and focus on how to modify the body of the list. A `cur` node, initialized as the `head` node before we enter the loop, will always be the current node that we are working on.

2. **Decide the break point for the loop**

   This will usually be when our `cur` node reaches `NULL`, but exceptions exsist. For example, if we referenced`cur->next->value` in the loop, we must break out of the loop when `cur->next` is `NULL`.

3. **Handle edge cases**

   The `head` node usually needs to be handled seperately.