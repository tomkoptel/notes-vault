#algorithm 
Bubble Sort is known as quadratic sorting algorithm with $O(n^2)$ runtime. The algorithm relies on double pass. The outer loop starts from the last index and moves down to 0. The inner loops starts from 0 and moves up to the outer index value. For example, we have 3 elements to sort.
* outer index = 3 inner index = 0
* outer index = 3 inner index = 1
* outer index = 3 inner index = 2
* outer index = 2 inner index = 0
* outer index = 2 inner index = 1
* outer index = 1 inner index = 0
As we traverse the inner loop we move the biggest closer to the end. This similar to the bubble raising within water.
In case of the partially sorted collection we can exit earlier, if `no` swaps has been detected within and inner loop.
```kotlin
fun <T : Comparable<T>> ArrayList<T>.bubbleSort() {
  if (size < 2) return
  for (end in lastIndex downTo 1) {
    var swapped = false
    for (current in 0 until end) {
      if (this[current] > this[current + 1]) {
         Collections.swap(this, current, current + 1)
         swapped = true
      }
    }
    // avoids redundant passes 
    // for [1, 2, 3, 5, 4] it will perform one pass 
    // and skips all passes for [1, 2, 3]
    if (!swapped) return 
  }
} 
```