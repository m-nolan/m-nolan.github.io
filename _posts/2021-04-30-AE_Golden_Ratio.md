# Fun Find: Golden Ratio in Transform Design

Michael Nolan

2021.04.29

A lot of my research uses a machine learning architecture called an [autoencoder](https://en.wikipedia.org/wiki/Autoencoder), which is a pair of neural network modules called an *encoder* and *decoder* which seek to transform an input sample back to itself. This might seem silly at first glance, but the reason for using this kind of model is that one can develop useful embedded representations of their data in the space between the encoder and decoder. This is known as the *embedded* or *latent* space of an autoencoder model. This can be used for a number of applications, and I recommend you take a look at the wikipedia article I linked to above to learn more about them. Two common use cases are (1) to find a nonlinear dimensionality reduction of the model akin to a more expressive PCA and (2) to embed in a higher dimensional space to enable untangling and koopman-esque representation of complex data distributions. My work makes use of the latter to reconstruct brain wave recordings, but this model type works on a wide range of data types. It's an impressive and flexible unsupervised machine learning technique and one of my favorite bits of mathematical machinery.

When designing an autoencoder, it's important to choose the right size embedded dimension. The number of dimensions modeled in the model's latent space can add up to important reconstruction objective boosts, but come at a cost of increased parameter counts and larger memory footprints. This tradeoff makes optimizing the sizes of the embedded spaces in your machine learning models a matter of both performance and practicality. Playing around with this kind of problem, I found a nice simple example of this very practical problem with a very pretty solution.

Consider a rather contrived example of an autoencoder model composed of two RNN units, one in the encoder and one in the decoder. After some experimentation and a few hyperparameter sweeps, you find that an asymmetric design performs just as well as a symmetric design: given your reconstruction threshold, the size of your decoder hidden state can be considerably smaller than your encoder. This is great! You'll need to add some linear transforms inbetween the RNN cells to make everything line up, but this could save you a good amount of RAM. Nice!

*So here's the question*: considering the encoder hidden state transfer matrix $U_e \in R^{n\times n}$ and a decoder hidden state transfer matrix $U_d \in R^{m\times m}$ with a linear tranform between them $U_t \in R^{n \times m}$, how much of a decrease in the decoder size must be made before you are actually using fewer parameters than you would be with an equidimentional model? In other words, solve for the ratio $\frac{n}{m}$ that satisfies the following:

$$2n^2 > n^2 + n m + m^2$$

Note that in the equidimensional case, there is no need for a linear tranform to match dimensions between the encoder and decoder.

Simplifying the equation algebraically, we get the following:

$$n^2 > n m + m^2 \rightarrow n^2 - nm - m^2 > 0$$

Dividing both sides by $m^2$ and letting $x := \frac{n}{m}$ we get:

$$\left(\frac{n}{m}\right)^2 - \frac{n}{m} - 1 > 0 \rightarrow x^2 - x - 1 > 0$$

...does this look familiar? If so, good eye! I didn't see it until working through the quadratic equation to find the roots of this inequality, but this is one form of the definition for the [**GOLDEN RATIO**](https://en.wikipedia.org/wiki/Golden_ratio)! It's an important number in a few fields, most visually in classical design and aesthetics, but it's notable among irrational numbers for being [the most irrational of them all](http://www.ams.org/publicoutreach/feature-column/fcarc-irrational4)! Please refer to that page for relevant figures, but know that this ratio defines a rectangle with side lengths $a, b \in R^+$ such that $b > a$ and:

$$\frac{b}{a} = \frac{b + a}{b}$$

Multiplying both sides by $a b$ you get the following:

$$b^2 = a b + a^2 \rightarrow b^2 - a b - a^2 = 0$$

Adopting the common notation for this ratio, let $\phi = \frac{b}{a}$ and reduce the equation by dividing both sides by $a^2$:

$$\left(\frac{b}{a}\right)^2 - \frac{b}{a} - 1 = 0 \rightarrow \phi^2 - \phi - 1 = 0$$

Therefore, given our equations defining $x$ and $\phi$, it follows that $x > \phi$. In other words, you're saving precious RAM so long as you set up your autoencoder with the following dimension restraint: $n > \phi m$.

I don't know if this knowledge is worth its weight in gold, but it sure is fun to see a classical irrational number fall out of an otherwise very dry optimization-for-hardware-reasons problem.
