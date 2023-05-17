#data-structure 
FIFO - first in first out or Queue.
LIFO - last in first out or Stack.
# ArrayList based implementation
Dequeue the element from the arraylist requires us to shift all elements from the head by 1, thus additions is $O(n)$.
Enqueue has amortisized `O(1)` as we add to the end, we simply write to the underlying empty space.
The space complexity is `O(n)` . As soon as we reach the capacity we need to double. Doubling the size of the underlying array will have a negative impact on the memory footprint.
# Doubly Linked List based implementation
Both dequeue and queue operation take $O(1))$ as we simply remove the node from head or add a new node to the tail. The main disadvantage of this solutions is that we need to allocate additional objects and those objects are not contiguous in the memory (or spatial locality) elements close to each other in memory.
The space complexity is $O(n)$.
# Ring Buffer based implementation
Has benefits of both $O(1)$ for queue and dequeue operations. Its space complexity is `O(n)`. Its disadvantage is a fixed size restictions. We are working with random access array behind the scenes and wrap over the last index following the formulat of `newIndex = (index + 1) % capacity` (see [[Why 9 % 10 is 9?]]]). The references are contiguous in the memory.
# Double-Stack based implementation
We keep 2 stacks behind the scene. The right stack used to store elements we get from queue call. The left stack populated each time when we either call peek or dequeue, only and only if left stack is empty. This makes peek and dequeue and amortisized `O(1)`.

# Reverse Queue
To reverse the queue we can first dequeue everything to stack and then pop everything from it back to queue. For example, queue of 1>2>3>4>5 will end up 5>4>3>2>1 in stack. The stack pops from the head, thus it will populate the second queue retaining the priority of the top most having the highest prirority.
