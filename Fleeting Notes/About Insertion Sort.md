#algorithm 
The more the data sort the less insertion sort needs to do thus making it the best algorithm to choose between bubble and selection sort.
The best case will give us $O(n)$.
The worst and average case will give us $O(n^2)$.
For the small lists it is most efficient in comparison between bubble, selection and insertion sorts. The same can be sad about the partially sorted data set.

The trick of algorithm is to increase the subset we are looking into for each next iteration. For `n` size data set we need perform reads for the subset of `(n..1)`.
Let say we have 5 elements than respective iterations will look like.
```
1: 1-1
	1': access el at 1 and 0 then compare, swap
2: 2-1
	2': access el at 2 and 1 then compare, swap
	2'': access el at 1 and 0 then compare, swap
3: 3-1
	3': access el at 3 and 2 then compare, swap
	3'': access el at 2 and 1 then compare, swap
    3''': access el at 1 and 0 then compare, swap
4: 4-1
.   4': access el at 4 and 3 then compare, swap
	4'': access el at 3 and 2 then compare, swap
	4''': access el at 2 and 1 then compare, swap
    4'''': access el at 1 and 0 then compare, swap
```

```kotlin
fun <T : Comparable<T>> MutableList<T>.insertionSort() {
	if (this.size < 2) return
	for (current in 1 until size) {
	   for (shift in (1..current).reverse()) {
	      if (this[shift] < this[shift - 1]) {
	         Collections.swap(this, shift, shift-1)
	      }
	   }
	}
}
```