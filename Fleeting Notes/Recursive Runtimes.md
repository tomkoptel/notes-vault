#big-o 
Recursive algorithms produce trees. For example, the fibonaci sequence of N will be:
```java
int fib(int n) {
 if (n <= 1) return 1;
 return f(n - 1) + f(n - 1);
}
```
The tree will have a depth of N. The function makes 2 calls and can be described as each of the levels as we drill down to `N` == 1 . For the example, if we call `fib(4)` we will have $2^4$ calls or runtime will be $O(n^2)$.
We can generalize the approach for any recursive function that makes multiple calls the runtime will often be $O(branches^[depth])$.