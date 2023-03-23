#algorithm #series
The binary search operates on the principle of dividing the original collection of N by 2 every pass. 
What is the runtime?. The total runtime is a number of steps we can make until N becomes. On each step we divide N by 2. If looked in reverse: "How many multiplies taken until we get N?"
$$
\begin{align*}
N_0 = 16\\
N_1 = 8\\
N_2 = 4\\
N_3 = 2\\
N_4 = 1\\
\end{align*}
$$

```kotlin
class Solution {  
    fun search(nums: IntArray, target: Int): Int {  
        val s = nums.size  
        if (s == 0) return -1  
        var upperBound = s - 1  
        var lowerBound = 0  
        while (lowerBound <= upperBound) {  
            val mid = (lowerBound + upperBound) / 2
            val element = nums[mid]  
            if (element == target) {  
                return mid  
            } else if (element < target) {  
                lowerBound = mid + 1
            } else {  
                upperBound = mid - 1
            }  
        }  
        return -1  
    }  
}
```

When searching `154` in `1 to 100`
```
before=0 lowerBound=0 upperBound=98 mid=49
after=0 lowerBound=50 upperBound=98 mid=49

before=1 lowerBound=50 upperBound=98 mid=74
after=1 lowerBound=75 upperBound=98 mid=74

before=2 lowerBound=75 upperBound=98 mid=86
after=2 lowerBound=87 upperBound=98 mid=86

before=3 lowerBound=87 upperBound=98 mid=92
after=3 lowerBound=93 upperBound=98 mid=92

before=4 lowerBound=93 upperBound=98 mid=95
after=4 lowerBound=96 upperBound=98 mid=95

before=5 lowerBound=96 upperBound=98 mid=97
after=5 lowerBound=98 upperBound=98 mid=97

before=6 lowerBound=98 upperBound=98 mid=98
after=6 lowerBound=99 upperBound=98 mid=98

-1
```
When searching `88` in `1 to 100`
```
before=0 lowerBound=0 upperBound=98 mid=49
after=0 lowerBound=50 upperBound=98 mid=49

before=1 lowerBound=50 upperBound=98 mid=74
after=1 lowerBound=75 upperBound=98 mid=74

before=2 lowerBound=75 upperBound=98 mid=86
after=2 lowerBound=87 upperBound=98 mid=86

before=3 lowerBound=87 upperBound=98 mid=92
after=3 lowerBound=87 upperBound=91 mid=92

before=4 lowerBound=87 upperBound=91 mid=89
after=4 lowerBound=87 upperBound=88 mid=89

before=5 lowerBound=87 upperBound=88 mid=87
87
```