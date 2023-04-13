Given the snippet:
```java
public class Main {  
    public static void printUnorderedPairs(String[] argsA, String[] argsB) {  
        for (int i = 0; i < argsA.length; i++) {  
            for (int j = 0; j < argsB.length; j++) {  
                System.out.println(args[i] + args[j]);  
            }  
        }  
    }  
}
```
The inner loop job is $O(1)$. We for each array A we make B number of calls of array B. It means that we need multiply. This leads to `O(a*b)`. Not to be confused with `O(n^2)` because the size of array A and array B matters.