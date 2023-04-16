#algorithm #series #big-o 
Is a concept that counts both the worst case of the runtime execution and common time spent on the runtime.
For example, the ArrayList doubles its capacity each time we reach the limit. Most of the time insertion will be constant `O(1)` , but in the worst case we will spend O(2X) time. The `Amortized time` means that when the worst case scenario happens it is the moment of the data structure `amortisizing` itself to enable for the most time the most efficient computation. 

What is the rumtime of the worst case scenario (the doubling of the size)? For that we need to compute sum up to X where geometric series is (1 + 2 + 4 + 8 + ... + X). This yields us geometric series X + X/2 + X/4 + ... + 1 also (1 + 2 + 4 + 8 + ... + X). The sum of the geometic sequence will be roughly 2X. So the space complexity is roughly O(2X). 

Mathematically it can be proven by relying on the formula of geometric series sum.
$$

S_n=\frac{a(1-r^n)}{1 - r}

$$
Where `S` - is the sum of series, n - is the number of terms (in our case it is what we are looking for), `a` - first term, `r` - ratio. X from above determines us the number of terms in the series. Let's substitute values we have `a = 1` , `r = 2`
$$
\begin{align*}
Sn(1-r)=a(1-r^n)\\
Sn(1-2)=1(1-2^n)\\
-Sn=1-2^n\\
Sn=2^n-1\\
\end{align*}
$$
In our problem statement X is <u>upper limit of the sum</u>, thus we can replace $S_n = S_x$. The sum of the terms up to X(X not inclued) is denoted with $S_x$. $S_n$ is used when we **know** the number of terms in the sequence. We are summing to a specific term X, not a specific number of terms.
$$
S_x = 2^x - 1
$$
For example, 1 + 2 + 4 + 8 + 16 where <u>16 is our X</u> then $S_4=1+2+4+8=15$ or $S_4=2^4-1=16-1=15. The amount of memory we use perform binary search on 16 elements will be sum of $S_4$ and 16 which gives us 15 + 16 or roughly 2X.