#big-o #series 
What is a Big-O of `printUnorderedPairs`? We are performing N iterations in the outer loop. For the first step we make (n-1) steps, for second (n-2) and so on. Summing all the steps we did in the inner loop we need to fallback to the arithmetic series.
The sum of steps done inside inner loop we get the following arithmetic series.
$$(n - 1) + (n - 2) + (n - 3) + ... + 2 + 1 + 0$$
According to the sum of arithmetic series formula:
$$S_n = \frac{n}{2}((2a + (n - 1)d) $$
Where `d=1` and `a=0` 
$$S_n = \frac{n(n-1)}{2} = \frac{n^2 - n}{2}$$
The main term is $n^2$ thus the Big-O will be $O(n^2)$.
```java
public class Main {  
    public static void printUnorderedPairs(String[] args) {  
        for (int i = 0; i < args.length; i++) {  
            for (int j = i + 1; j < args.length; j++) {  
                System.out.println(args[i] + args[j]);  
            }  
        }  
    }  
}
```
Think about the number of pairs the snippet produces. Let's see the pairs produced to for n=5
$$
\begin{align*}
(0, 1), (0, 2), (0, 3), (0, 4) \\
(1, 2), (1, 3), (1, 4) \\
(2, 3), (2, 4) \\
(3, 4) \\
\end{align*}
$$
Which gives us 4 + 3 + 2 + 1 = 10 or if plugged in formula $$S_5 = \frac{25 - 5}{2}=10$$