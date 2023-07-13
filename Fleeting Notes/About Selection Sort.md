#algorithm 
Selection sort is also a quadratic algorithm that has worst average time complexity of $O(n^2)$. The best runtime is  $O(n^2)$, unlike in bubble sort. The bubble sort performs $O(n)$ operation for the pre-sorted array. Despite that selection sort performs better than bubble sort as it performs only $O(n)$ swaps. It is true because insertion sort only swaps the smallest element in the collection while bubble sort swaps every pair of the elements that are out of order and achieves $O(n^2)$.

* The key of insertion sort is to select the smallest (for ascending).
* Move the lowest to the sorted position.
* The selected element swapped with the first element of unsorted element.

```kotlin
fun <T : Comparable<T>> ArrayList<T>.selectionSort() {  
    if (size < 2) return

    for (current in 0 until lastIndex) {
      var lowest = current
      for (other in (current + 1) until size) {
		if (this[other] < this[lowest]) {
		   lowest = other
		}
      }

	  if (lowest != current) {
	      Collections.swap(this, lowest, current)
      }
    }
```
