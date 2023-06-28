#data-structure 

# Is Binary Search Tree?
One of the approaches is to recursively visit each branch by passing lower bound of current node when moving in the left direction and passing current value of node as an upper bound when moving in the right direction.
The Binary Search Tree follows the idea that each node inserted should conform to the boundaries imposed by the parent. When we are injecting node we check and the node is less than current, then we pass it to the left, which, in this case, means that we are having an upper bound (the new value is less than parent). When we are injecting node we check and the node is more than current, then we pass it to the right, which, in this case, means that we are having a lower bound (the new value is more than parent).
![tree invariant](../Images/tree-ivariant.png)
# Removing Node
Any removal starts with the search of the node that holds the value. As soon as we found the node we need to check:
1. Is the node a leaf of the tree (both left/right branches null).
	* If we came from the left direction (we need to keep the information of the direction). Then we set the left branch of the parent node to null, otherwise the right.
2. Is the node has left branch and `no` right branch.
	* The reference to the left branch then assigned to the parents either left or right branches depending from which branch we accessed the current node.
3. Is the node has right branch and `no` left branch.
	* The reference to the left branch then assigned to the parents either left or right branches depending from which branch we accessed the current node.
4. Is the node has both right and left branches.
	* Find the in-order successor. It is a minimum value of the right branch. The value of the right node is bigger than the value of node we are working with. The minimum value of the right branch will be bigger of any value of the left branch.
	* The in-order successor than use to swipe with the the current node. It happens following way.
		* The left branch of the in-order successor node will reference the left branch of the current node.
		* The right branch of the in-order successor node will reference the right branch of the current node.
		* The parent node reference to the current node is replaced by the reference to the in-order successor. As a result, we are overwriting the reference and thus removing the value from the tree.