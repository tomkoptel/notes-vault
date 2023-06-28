#data-structure 
### Why it is legal to do rotations?
AVL Tree is a balance binary tree that is built on the premise that we are working with the binary tree. The insertion relies on the "tree invariant". The "tree invariant" is fundamental property as we always insert the lowest value to the left or to the right if more.
For example, if we insert 5 then 6 the 6 > 5 goes to 5. Then insert 4 the 4 < 6 goes to the left. The invariant is 4 < 5 < 6 where 5 is a root. As we rotate the tree we are retaining the invariant.
![[tree-invariant.png]]
In the left tree we hold true D < B < E < A < C. The same condition holds true for the right D < B < E < A < C.
### Why we need to pick the most bottom unbalanced node?
The simple answer is: "It is more efficient". The bottom-most unbalanced node after the rotation will restore the balance of the tree the fastest way. Selecting different option would be inefficient and will lead to cascading effect of rebalancing attempts.
