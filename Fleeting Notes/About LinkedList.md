#algorithm #data-structure
The implementation of the `non-double linked` LinkeList involves a data structure called node. The node holds value and the reference to the next node. The node is a convenient data strucuture. One is used to move forward in the LinkedList. 

```kotlin
data class Node<T : Any>(val value: T, var next: Node<T>? = null)
```

There are two known methods to traverse the linked list:
* slow - when we jump to the next reference within single pass of `while-loop`
* fast - when we jump to the next of the next reference the one is known as a `runner technique` (see find the middle item)
The node used to represent the tail and head of the `LinkedList`. When we append the value to the `LinkedList` we do modify the `tail` when we prepend the value we do modify the `head`.
Lets say we have `1 -> 2 -> 3 -> 4` the head will be `1` the taill will be `4`.

Let's create list using push. This will produce us `4 -> 3 -> 2 -> 1`
* 0: head=1 head.next=null tail=1
* 1: head=2 head.next=1 tail=1
* 2: head=3 nead.next=2 tail=1
* 3: head=4 nead.next=3 tail=1

Let's create list using add. This will produce us `1 -> 2 -> 3 -> 4`
* 0: head=1 head.next=null tail=1
* 1: head=1 head.next=2 tail=2
* 2: head=1 head.next=2 tail=3
* 3: head=1 head.next=2 tail=4

```kotlin
class LinkedList<T : Any> {
  private var head: Node<T>? = null
  private var tail: Node<T>? = null
  private var size = 0

  fun push(value: T) {
    head = Node(value, next = head?.next)
	if (tail == null) {
		tail = head
	}
	size++
  }

  fun add(value: T) {
    val tail = tail
    if (tail == null) {
	    push(value)
    } else {
	    val node = Node(value)
	    // on the second step `head == tail` so we essentially connect head to 
	    // the next tail, in all other steps we manipulate the previous tail
	    tail.next = node
	    tail = node
    }
  }
}
```

> The best way to think about nodes is to ask youself what is the state of the linked list we are working with at the moment. At initial stage we have `head==tail`. Any other manipulations is about the previous and next node incoming.

# How to find the middle element?
As mentioned above we can deploy the `runner technique`. First we have a slow and then fast references. Then we traverse each of the node fast reference twice per iteration step. The Big O will be O(n/2) or O(n), because we disregard the constant factor for the infinitevly big data set.
```kotlin
fun middle(): Node<T>? {
	var slow = head
	var fast = head

	// 1 -> 2 -> 3 -> 4 -> 5
	while(fast != null) {
		fast = fast.next
		// this for the case when the list contains 0 or 1 items
		// we need check for even numbers to achieve (n / 2) + 1 as middle item
		if (fast != null) {
			slow = slow?.next
			fast = fast.next
		}
	}

	return slow
}
```
# How to revert the LinkedList?
There are 2 ways we can approach it. First is a recursive way, which due to the heap limitation is not the perfect, but simplest to reason about approach.
```kotlin
fun <T : Any> recursiveRevert() {
	val newList = LinkedList<T>()
	drillDown(head, newList)
	return newList
}

private fun <T: Any> drillDown(node: Node<T>, list: LinkedList<T>) {
	val next = head.next
	if (next != null) {
		drillDown(next, list)
	}
	list.add(node.value)
}
```
The second is iterative approach where we are de-link one node.
* 1 -> 2 -> 3 -> 4  
* step=0 current=1 -> 2 -> 3 -> 4 prev=null  
* step=1 current=2 -> 3 -> 4 prev=1  
* step=2 current=3 -> 4 prev=2 -> 1  
* step=3 current=4 prev=3 -> 2 -> 1  
* 4 -> 3 -> 2 -> 1

```kotlin
fun reverse() {
	var current = head
	// keep the reference to the head, it will become the tail in the end
	val nextTail = head
	var prev: Node<T>? = null
	
	while (current != null) {
		// grab the next node
		val next = current.next
		// assign the reference of the previous iteration
		current.next = prev

		// the current node is detached now and we need attach it again later
		prev = current
		// now the current will be the next item
		current = next
	}

	head = prev
	tail = nextTail
}
```
# Merge to sorted lists into the one presrving order
The time complexity will be $O(L + R)$ where L-size of the left list and R-size of the right list.

```kotlin
fun <T: Comparable<T> LinkedList<T>.sortedMerge(other: LinkedList<T>): LinkedList<T> {
	var left = nodeAt(0)
	if (left == null) return other 
	var right = other.nodeAt(0)
	if (right == null) return this
	
	val newList = LinkedList<T>()

	// continues until one of the loops reaches its end
	while(left != null && right != null) {
		if (left.value < right.value) {
			left = left.appendTo(newList)
		} else {
			right = right.appendTo(newList)
		}
	}

	while (left != null) {
		left = left.appendTo(newList)
	}
	
	while (right != null) {
		right = right.appendTo(newList)
	}

	return list
}

private fun Node<T>.appendTo(list: LinkedList<T>): Node<T> {
	list.add(value)
	return next
}
```