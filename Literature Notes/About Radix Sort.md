#algorithm 
Notes based on [[Data structures & algorithms in Kotlin, I. Galata, M. Braun, M. Suica]] page 336.
* The radix sort is `non-comparative` algorithm. It means that in order to sort we need to group items instead of comparing each one by one.
* The radix sort takes advantage of the numbers structure. It relies on the iterative analysis of digits position within number.
	* MSD - least significant division based on the digit position analysis starting from right side (e.g 1234 will be analysed as 4, 3, 2, 1).
	* LSD - most significant division based on the digit position analysis starting from left side (e.g 1234 will be analysed as 1, 2, 3, 4).
* The asymptotic complexity of the radix sort is `O(k*n)` where `k` is the number of significant digit of the largest number (e.g 132, 11, 1235 the `k=4`) and `n` is number of digits in the collection.

# Implementation of MSD radix sort
1. For each iteration we loop through numbers one by one starting from right.
2. We keep reference to the `base=10` and `digits=i` where is `i=1` at first iteration.
3. We multiply the `digits` each time by base (e.g {0, 1}, {1, 10}, {2, 100}). This allows us to focus on the significant part of number.
	1. We exit the loop, only and only if, the `remainingPart = number / digits` is more than `0`.
4. For the number 12345 on the 2 iteration we will have 123 then we take modulus and get our grouping digit `3`.
5. We create list of 10 lists(buckets). For each, digit we populate the bucket with the number.
6. After the grouping is complete we flatten all numbers from the bucket list down to the current collection.
# Implementation of LSD radix sort
1. We need a function that takes list and computes the max number most significant digit position.
2. To compute the most significant digit.
	1. Divider the number by 10 until result of division is more than 0.
	2. Increment temporary `i` by 1 each time we succeed with the iteration.
3. We need to create a stack that holds pair of the current list and position. The position represents current most significant number. We then increase a position for each entry of the list. For the example, the number of 1353 we will first access {0, 3}, {1, 5}, {2, 3}, {3, 1}.
4. We loop our stack until empty.
	1. Pop instance of the list and position.
	2. We grab the number of most significant digit position and compare to the current position.
	3. If position is more than most significant digit position we add all numbers to the result list.
5. If position is less than we create bucket list of 10 lists.
6. We group the digits by position into the bucket list.
7. Then from 9 until 0 we populate stack with non empty bucket and position increased by 1.
The key here is that stack allows us to perform breadth first traversal without relying on the recursion that exhausts the stack.

>In summary, radix sort exploits the inherent numeric/character-based order in data by grouping and sorting on individual digit/character positions sequentially from least to most significant. This allows it to avoid comparisons and gives it efficient runtime on large datasets.