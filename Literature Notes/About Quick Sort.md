#algorithm 
Notes based on [[Data structures & algorithms in Kotlin, I. Galata, M. Braun, M. Suica]] page 361.
According to I. Galata, M. Braun and M. Suica the merge sort is preferable over quicksort when you need stability. The stability is about preservance of the relative order of elements with equal values. Merge sort is a stable sort and guarantees $O(n*log(n))$. For quicksort when the pivot is poorly selected, then runtime can degrade up to $O(n^2)$. Merge sort works better for larger data structures where elements scattered through memory. Quicksort performs better for the contiguous block of memory and as it performs less moves to move more data around.

Quick sort focuses on dividing the list into sublists and rearranging elements based on the pivot selection. The focus of the techniques is to pick the right pivot. There are 3 major partition techniques:
* picking element in the middle
* picking last element Lomuto's method
* picking first element Hoarse method
Neither works perfect and will produce like an insertion sort for the worst case $O(n^2)$ runtime. The selection of the pivot can be improved if we first rebalance the elements and then perform sorting either with Lomuto or Hoarse method. It is known as media of three and its idea is to swap low, middle and high index elements to balance out the original collection.
```kotlin
// [14, 6, 8, 7, 9]
fun <T: Comparable> List<T>.medianOfThree() {
  if (this[low] > this[center]) {
	  Collections.swap(this, low, center)   // [8, 6, 14, 7, 9]
  }
  if (this[low] > this[high]) {
     Collections.swap(this, low, high)   // [8, 6, 14, 7, 9]
  }
  if (this[center] > this[high]) {
	  Collections.swap(this, center, high) // [8, 6, 9, 7, 14]
  }
  return center
}
```
The mentioned approach media of three lacks in effectively handling duplicates. To account for the duplicates there is also exist Dutch national flag partitioning.
## Lomuto's Partition
```kotlin
fun <T: Comparable> List<T>.lomutoPartition(low: Int, high: Int) {
  val pivot = this[high]
  var i = low
  for (j in low until high) {
	if (this[j] <= pivot) { // was mistake
		Collections.swap(this, i, j)
		i++
	}
  }
  Collections.swap(this, i, high)
  return i
}
```
## Hoare Partition
```kotlin
fun <T: Comparable> List<T>.hoareSort(low: Int, high: Int) {
  val pivot = this[low]
  var i = low - 1
  var j = high + 1
  while (true) {
	do {
	  i++
	} while (this[i] < pivot)
	do {
		j--
	} while (this[j] > pivot)
	if (i < j) {
		Collections.swap(this, i, j)
	} else {
		return j
	}
  }
}
```
## Dutch Flag Quick Sort
```kotlin
fun <T: Comparable> List<T>.hoarseSort(low: Int, high: Int): Pair<Int, Int> {
	var lowest = low
	var equal = low
	var highest = high
	while (equal <= highest) {
		if (this[equal] < pivot) {
			Collections.swap(this, lowest, equal)
			lowest++
			equal++
		} else if (this[equal] == pivot) {
			equal++
		} else {
			Collections.swap(this, equal, highest)
			highest--
		}
	}
	return (lowest to highest)
}
```
## The core of the Quick Sort
The general algorithm performs recursive sort. The difference lies in the way the next partition defined.
* Lomuto's case already placed the pivot in its center position thus we sort before and after pivot
* Hoare's case requires us to include pivot when sorting the left part.
* Dutch Flag case is similar to Lomuto because the pivot elements where already swapped
```kotlin
fun <T : Comparable<T>> List<T>.quickSort(): List<T> {  
    if (size < 2) return this  
    val stack = LinkedList<Pair<Int, Int>>()  
    stack.push(0 to size - 1)  
    while (stack.isNotEmpty()) {  
        val (low, high) = stack.pop()  
        if (low < high) {
          val pivotIndex = medianOfThree(low, high)  
		  Collections.swap(this, pivotIndex, high)
		  // Lomuto
		  val pivot = lomutoPartition(low, high)
		  stack.push(pivot + 1, high)
		  stack.push(low, pivot - 1)
		  // Hoarse
		  val pivot = hoareSort(low, high)
		  stack.push(pivot + 1, high)
		  stack.push(low, pivot)
		  // Dutch Flag
		  val (lower, larger) = hoareSort(low, high)
		  stack.push(lower + 1, high)
		  stack.push(low, larger - 1)
        }
	}
```