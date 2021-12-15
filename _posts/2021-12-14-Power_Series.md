# Looking for Convergence: Nth Order Power Series

Michael Nolan

2021.12.14

This past quarter I was a teaching assistant for an undergraduate signal processing class. 
The course covered discrete and continuous time signal processing theory. 
I was tasked with writing homework assignments for the students.
Writing good DSP questions is tough. Time-domain problems (e.g. convolutions, fourier transforms) invariably result in infinite summations, which only converge in a relatively small number of cases.

I was writing these assignments for undergraduate engineering students taking a course that wasn't designed to stretch their brains over sequences and series material, so I narrowed that set down even further to make most every sum look like a geometric series.
For those that don't remember, a geometric series has the following form $\sum_{n=0}^{\infty}a^n = \frac{1}{1-a}\forall a: |a|<1$.
For example, a kindly considered fourier transform problem would look something like the above where $a = z e^{-j\omega}$ where $j=\sqrt{-1}$.
Down with $i$, long live $j$.

Kindness gets boring after a while.
I threw them a few problems involving trickier summations toward the end of the quarter.
One example was to compute the fourier transform of a system impulse resonse function $h[n] = n b^n u[n] |b|<1$ where $u[n]$ is the discrete heaviside step function. 
That computation looks like the following:
$$H(\omega) = \mathcal{F}\{h[n]\} = \sum_{n=-\infty}^{\infty}h[n]e^{-j\omega n} = \sum_{n=0}^{\infty}n b^n e^{-j\omega} = \sum_{n=0}^{\infty}n\left(b e^{-j\omega}\right)^n$$.

The last simplification comes from knowing that $u[n]$ is 0 for all $n < 0$. Also, the $n=0$ term will be zero because of $n$.

This summation now has the slightly more complicated form of $\sum_{n=1}^{\infty}n a^n$ where $a = b e^{-j\omega}$. 
[A little digging on wikipedia](https://en.wikipedia.org/wiki/List_of_mathematical_series#Power_series) will show you that this summation converges and is equal to $\frac{a}{(1-a)^2}$. Listed below that are some interesting identities for higher-order power series and their relationship to the [polylogarithm function](https://en.wikipedia.org/wiki/Polylogarithm). The sequence goes as follows, again for $|a| < 1$:
- $\sum_{n=1}^{\infty}n^2 a^n = \frac{a(a+1)}{(1-a)^3}$
- $\sum_{n=1}^{\infty}n^3 a^n = \frac{a\left(a^2 + 4a + 1\right)}{(1-a)^4}$
- $\sum_{n=1}^{\infty}n^4 a^n = \frac{a\left(a^3 + 11a^2 + 11a + 1\right)}{(1-a)^5}$

It looks like there's a pattern here! Let's see if we can suss it out.
Let's define the nth order power series as $S_N = \sum_{n=1}^{N}n^N a^n$.
The pattern above appears to be the following: $S_N = \frac{1}{(1-a)^(N+1)}\sum_{k=0}^{N-1}c_k a^{N-k}$.
Those sequences of $c_k$ values were foreign to me, but they appeared to follow a triangular pattern similar to but clearly distinct from the more familiar [Pascal's Triangle](https://en.wikipedia.org/wiki/Pascal%27s_triangle).
Some more googling and digging through wolfram mathworld pages got me an answer: the $c_k$ values were forming an [eulerian sequence](https://en.wikipedia.org/wiki/Eulerian_number) in each summation. 
The N-choose-kth eulerian number is equal to the number of permuations of the sequence $\{0,1,2,\dots,N-2,N-1\}$ containing exactly $k$ "runs", or subsets of adjacent elements that are strictly increasing.
This threw me for a loop. If you're confused about how that definition relates to our larger power series question, know you're in good company.
For the analysis at hand, knowing the connection is not necessary; however, know that you can find [more information here](https://mathworld.wolfram.com/EulerianNumber.html).




