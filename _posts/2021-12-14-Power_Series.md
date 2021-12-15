# Looking for Convergence: Nth Order Power Series

Michael Nolan

2021.12.14

This past quarter I was a teaching assistant for an undergraduate signal processing class. 
The course covered discrete and continuous time signal processing theory. 
I was tasked with writing homework assignments for the students.
Writing good DSP questions is tough. Time-domain problems (e.g. convolutions, fourier transforms) invariably result in infinite summations, which only converge in a relatively small number of cases.

I was writing these assignments for undergraduate engineering students taking a course that wasn't designed to stretch their brains over sequences and series material, so I narrowed that set down even further to make most every sum look like a geometric series.
For those that don't remember, a geometric series has the following form $\sum_{n=0}^{\infty}a^n = \frac{1}{1-a}\forall a: |a|<1$.
For example, a kindly considered fourier transform problem would look something like the above where $a = z e^{-j\omega}$ where $j=\sqrt(-1)$.
Down with $i$, long live $j$.

Kindness gets boring after a while.
I threw them a few problems involving trickier summations toward the end of the quarter.
One example was to compute the fourier transform of a system impulse resonse function $h[n] = n b^n u[n] |b|<1$ where $u[n]$ is the discrete heaviside step function. 
That computation looks like the following:
$$H(\omega) = \mathcal{F}\{h[n]\} = \sum_{n=-\infty}^{\infty}h[n]e^{-j\omega n} = \sum_{n=0}^{\infty}n b^n e^{-j\omega} = \sum_{n=0}^{\infty}n\left(b e^{-j\omega}\right)^n$$
The last simplification comes from knowing that $u[n]$ is 0 for all $n < 0$. Also, the $n=0$ term will be zero because of $n$.

This summation now has the slightly more complicated form of $\sum_{n=1}^{\infty}n a^n$ where $a = b e^{-j\omega}$.
