#math
The answer lies in the definition of [Euclid's division lemma](https://en.wikipedia.org/wiki/Euclidean_division)  
$$a = b*q + r$$
where $r >= 0$ and $r < b$ .
In our case a = 9, b = 10.
$q=0$ because it can not be $q > 0$. Why? If it becomes bigger than zero we will end up with $b*q >= b$ it follows that $b*q > a$ . We know that $a < b$ thus if $q > 0$ we get a situation that $b*q > a$ which contradicts $a = b*q + r$ . Inequality here makes the contradiction with the equality.
