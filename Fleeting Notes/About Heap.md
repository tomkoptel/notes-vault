#data-structure 
Is perfect binary tree tha trelies on the comaprable properties of the structure that is contained. If the comparison betwee item A and B returns value:
* less than 0 means that A < B
* more than 0 means that A > B
* equals 1  means that A == B
With the help of different implementation of comapartor we can implement 2 strategies:
* heap that stores minimum value at the root -> minheap
* heap that stores maximum value at the root -> maxheap
![[heap-bigo.png]]
# Insert
As we insert items we do inject one in the end of the underlying collection. Then by traversing each parent we move the element to the position matching the invariant defined by comparator.
For the example, in the max heap the item will move up, if and only if the value is bigger than its direct left/right parent value. The operation of moving element up called sifting up.
# Remove
1. If the element we remove is not last element swap the current element with the last element of the collection.
2. The sift down the element and then sift up the element.
Sift down performs lookup of the left and right children, if the element does not respect an invariant we move it down to the node with the best matching value.
# Search
In case of search we need traverse both right and left branches. This can lead in the worst case to `O(n)`.
# Indexes
* Left index `(2 * idex) + 1`
* Right index `(2 * idex) + 2`
* Parent index `(index - 1) / 2`

![[heap-left-right-index.png]]

![[heap-as-array.png]]