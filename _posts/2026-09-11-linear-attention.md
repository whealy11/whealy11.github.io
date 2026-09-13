---
layout: default
title: Linear Attention From Scratch
description: This builds all the theory required to understand linear attention from first principles. It requires no prerequisite knowledge of kernels.
category: technical
image: /assets/images/linear-attention.jpg
---

# Linear Attention from first principles

**Will Healy**
*September 2026*

## Introduction

The goal of this piece is to build a deep understanding of linear attention. I will assume the reader is equipped with a basic understanding of attention, linear algebra, calculus, and statistics, but nothing more. No knowledge of kernels is necessary.

## 1. Attention: what is non-linear to begin with?

Attention is a mechanism for computing and aggregating importance weights for components in a system. It became popularized by the paper *[Attention Is All You Need](https://arxiv.org/abs/1706.03762)*, as it proved to be affective in aggregating information in sequences of text. The formula for attention is the following. *Please note that my notation differs from typical literature, where A is usually defined to be the product of Q and $$K^T$$, not the output of the attention mechanism.*

$$
A = \operatorname{softmax}\left(\frac{QK^T}{\sqrt d}\right)V
$$
### Dimensions

The dimensions of the matrices are the following:

$$
Q \in R^{n\times d}
$$

$$
K \in R^{n\times d}
$$

$$
V \in R^{n\times d_v}
$$

where $n$ is the number of tokens in our context window (often called our sequence length) and $d$ is the dimensionality of the embeddings for each token.

$d_v$ can differ from $d$, but many implementations have them equal. We will assume they're equal for simplicity.



The i'th row of Q, K, and V represent the i'th token in the sequence. I will let $x_i$ represent the i'th row of a matrix X and let $X_i$ represen thte i'th column of a matrix X. $q_i$, $k_i$, and $v_i$ are all linear transformations of some common embedding for the token at that index. Usually the common embedding is the sum of a fixed base embedding and a positional embedding. So

$$
q_i = (b_{token_i} + p_i)^T W_Q
$$

$$
k_i = (b_{token_i} + p_i)^T W_K
$$

$$
v_i = (b_{token_i} + p_i)^T W_V
$$

where $W_Q \in R^{dim(b) \times d}$ is the linear transformation, $b_{token_i}$ is the base embedding for the token at position i, and $p_i$ is the positional embedding for the ith position.

### Interpretation of A

Now let's consider the i'th row of A, $a_i$. $a_i$ is a contextually aware embedding of the i'th token. That means that it is an aggregation of its own token's value, the position of its own token, and the meaning and positions of the tokens surrounding it. In causal attention, which is what is most commonly used in LLMs, $a_i$ will only have context on itself and what came before it. We can write $a_i$ as the following:

$$
a_i =
\frac{
\sum_{j=1}^i
\exp\left(\frac{q_i^T k_j}{\sqrt d}\right)v_j
}{
\sum_{j=1}^i
\exp\left(\frac{q_i^T k_j}{\sqrt d}\right)
}
$$

Note what this is. This is a weighted average of the rows of V. The weight for the j'th row is equal to

$$
\exp\left(\frac{q_i^T k_j}{\sqrt d}\right)
$$

We then normalize by the sum of these weights.
### Runtime

Let's consider the runtime of this operation. For each of the n elements in the sequence, we need to compute a dot product between two $d$-dimensional vectors. Thus, the run time of this operation is $O(n*d)$. In an LLM, we would be doing this to get the contextually aware representation of the last token in the sequence, and then use this to predict the next token. To generate a sequence of length n, we would have to do this operation n times, which yields a runtime of $O(n^2d)$.

### Our Goal

Our goal with linear attention is to slash this runtime by a factor of n. This means that the cost to calculate a contextually aware embedding for an individual token is $O(d)$ and the cost to calculate it for an entire sequence is $O(nd)$. We will use kernel tricks to accomplish this.

## 2. Kernels

To slash the runtime by a factor of n, we will need to rely on kernels. This section will introduce the concept of a kernel, prove two useful theorems related to kernels, then use these theorems to introduce the kernels necessary to understand linear attention.

### Definition

A kernel $k: \chi \times \chi \to R$ is a function which maps a pair of elements from an arbitrary domain to a real number while maintaining an important property. The important property is that $k(x, x')$ can be expressed as

$$
\langle \phi(x), \phi(x') \rangle,
$$

where $\phi$ is an arbitrary mapping from $\chi$ to an element in some Hilbert $\mathcal{H}$ space and $\langle \cdot,\cdot \rangle$ is the inner product in $\mathcal{H}$. A Hilbert space is simply a potentially infinite dimensional vector space with one technical requirement. If this notation is intimidating, just think of $\langle \cdot,\cdot \rangle$ as being a dot product and $\phi$ to be some mapping from a domain to a vector space.

### Theorem 1: Kernels are Positive Semi-Definite

Kernels are positive semi-definite (PSD). PSD has a slightly different meaning when referring to kernels than when applied to matrices, but as we'll see in a moment the two definitions are closely related.

A matrix $X \in R^{n\times n}$ is PSD if it is symmetric and satisfies

$$
\forall v \in R^n:\quad v^T X v \geq 0.
$$

Note that this is the same as enforcing the non-negativity of $\langle v, Xv\rangle$ for all $v$ with the standard inner product. So geometrically it's saying that X doesn't transform any vector over the hyperplane defined by the set of points orthogonal to $v$. It's a multi-dimensional extension of non-negativity.

For kernels, PSD means that for any set of elements in $\chi$ of arbitrary size $k$, call it

$$
\{x_1, x_2, x_3, \ldots, x_k\},
$$

and any vector $v \in R^k$, it holds that

$$
\sum_{i=1}^k\sum_{j=1}^k v_i v_j k(x_i,x_j) \geq 0.
$$

At first glance this may look unrelated to the matrix definition of PSD, but consider some matrix $X$ and it's associated *Gram* or *Kernel* matrix $X^T X$. $X^T X$ is trivially PSD since

$$
v^T X^T Xv = (Xv)^T(Xv) = \langle Xv,Xv\rangle \geq 0
$$

by the definition of inner products.

Also note that

$$
(X^T X)_{ij} = X_i^T X_j
$$

where $X_i$ is the i'th *column* of X. So, this is called a kernel matrix since the (i, j) entry in the matrix is equal to $k(X_i,X_j)$ where the kernel:

$$
k(x,x') = x^Tx'
$$

is a simple dot product.

But if we chose an arbitrary kernel $k$, we could define a kernel matrix K where

$$
K_{ij} = k(X_i,X_j),
$$

and now the definition of the kernel being PSD is identical to this matrix being PSD.

#### Proof of Theorem 1

Take an arbitrary kernel

$$
k(x,x') = \langle\phi(x),\phi(x')\rangle.
$$

We want to show that

$$
\sum_{i=1}^k\sum_{j=1}^k v_i v_jk(x_i,x_j) \geq 0
$$

holds for an arbitrary set of $\{x_1, x_2, ... x_k\}  \in [\chi]^k$ and an arbitrary vector $v \in R^k$.

$$
\begin{aligned}
\sum_{i=1}^k\sum_{j=1}^k v_i v_j k(x_i,x_j)
&=
\sum_{i=1}^k\sum_{j=1}^k
v_i v_j
\langle\phi(x_i),\phi(x_j)\rangle \\
&=
\sum_{i=1}^k\sum_{j=1}^k
\langle v_i\phi(x_i),v_j\phi(x_j)\rangle \\
&=
\left\langle
\sum_{i=1}^k v_i\phi(x_i),
\sum_{j=1}^k v_j\phi(x_j)
\right\rangle \\
&=
\left\langle
\sum_{i=1}^k v_i\phi(x_i),
\sum_{i=1}^k v_i\phi(x_i)
\right\rangle
\geq 0.
\end{aligned}
$$

So our proof is complete. We have shown that any function expressible as the inner product between two outputs of a mapping from a domain to a Hilbert space is PSD.

### Theorem 2: All Symmetric PSD Functions are Kernels

This result is extremely counterintuitive, yet it is true. This says that any PSD function can be expressed as an inner product of some mapping to a Hilbert space. More explicitly, this shows the opposite direction of property 1.

Putting property 1 and 2 together we get the following bidirectional implication:

$$
\left(f:\chi\times\chi\to R\text{ is PSD}\right)
\longleftrightarrow
\left(f(x,x')=\langle\phi(x),\phi(x')\rangle\right)
$$

for some mapping $\phi$.

#### Proof of Theorem 2

We want to show that an arbitrary function

$$
f:\chi\times\chi\to R
$$

which satisfies

$$
\sum_{i=1}^k\sum_{j=1}^k
v_i v_jk(x_i,x_j)\geq0
$$

for arbitrary $v$ and ${x_1,x_2,\ldots,x_k}$, implies that

$$
f(x,x') = \langle\phi(x),\phi(x')\rangle
$$

for some mapping $\phi$.

First let's let our set ${x_1,x_2,\ldots,x_k}$ contain every element in $\chi$. If $\chi$ has infinite elements, then this is an infinite sized set. Since $f$ is symmetric PSD, we can imagine a potentially infinite dimensional matrix M such that

$$
M_{ij}=f(x_i,x_j).
$$

Since $f$ is symmetric PSD, $M$ is a symmetric matrix with non-negative eigenvalues. Thus, we can write

$$
M = Q\Lambda Q^T
= Q\Lambda^{\frac12}\Lambda^{\frac12}Q^T.
$$

If we let

$$
\Phi = (Q\Lambda^{\frac12})^T,
$$

we can write

$$
M=\Phi^T\Phi.
$$

Thus,

$$
M_{ij}=\langle\Phi_i,\Phi_j\rangle.
$$

This completes the proof, because we've shown that for all possible pairs $(x_i,x_j)\in\chi\times\chi$, it holds that

$$
f(x_i,x_j)=\langle\phi(x_i),\phi(x_j)\rangle,
$$

where $\phi(x_i)=\Phi_i$. Once again, $\Phi_i$ is the i'th column of $\Phi$ here.

### A few examples

Now let us understand three important kernels. These build the foundation of the theory behind linear attention, so they are worth understanding deeply. For all these examples, we can let $\chi=R^n$.

**1. Linear kernel**

$$
k(x,x')=x^Tx'
$$

is a kernel.

$$
\phi(x)=x
$$

and $\langle\cdot,\cdot\rangle$ is the standard dot product.

**2. Polynomial kernel**

$$
k(x,x')=(x^Tx')^m
$$

Since for $m=0$ and $m=1$ this is trivial, let's start with $m=2$.

$$
\begin{aligned}
(x^Tx')^2
&=
\left(\sum_{i=1}^d x_i x'_i\right)^2 \\
&=
\sum_{i=1}^d\sum_{j=1}^d
(x_i x'_i)(x_j x'_j) \\
&=
\sum_{i=1}^d\sum_{j=1}^d
(x_i x_j)(x'_i x'_j).
\end{aligned}
$$

This can be written as a dot product by defining a mapping

$$
\phi(x)
=
[x_1x_1,x_1x_2,\ldots,x_1x_n,x_2x_1,\ldots,x_2x_n,\ldots,x_nx_n].
$$

So, $\phi$ maps an n-dimensional vector into an $n^2$ dimensional vector. Once again we will still let $\langle\cdot,\cdot\rangle$ use the standard inner product.

Similarly, $(x^Tx')^m$ can be represented as an inner product, where $\phi$ maps out all the unique degree m monomials and their associated counts, thus mapping an n-dimensional vector to an $n^m$ dimensional vector.

**3. Exponential dot-product kernel**

$$
k(x,x')=\exp(x^Tx')
$$

To see how this can be written as a kernel, let us first write the Taylor series expansion of $e^x$. By Taylor series,

$$
e^z=\sum_{m=0}^{\infty}\frac{z^m}{m!}.
$$

Thus,

$$
e^{x^Tx'}
=
\sum_{m=0}^{\infty}
\frac{(x^Tx')^m}{m!}.
$$

Notice that this is a weighted sum of $(x^Tx')^m$, which we already showed was a kernel. Let's let

$$
k_m(x,x')=(x^Tx')^m.
$$

By theorem 2, we know that if we can show that $\exp(x^Tx')$ is PSD, then it must be a kernel.

In order for $\exp(x^Tx')$ to be PSD it must satisfy

$$
\forall v\ \forall\{x_1,\ldots,x_k\}:
\sum_{i=1}^k
\sum_{j=1}^k
v_i v_j
\left(
\sum_{m=0}^{\infty}
\frac{1}{m!}k_m(x,x')
\right)
\geq0.
$$

Rearranging the summations this can be rewritten as

$$
\sum_{m=0}^{\infty}
\frac{1}{m!}
\left(
\sum_{i=1}^k
\sum_{j=1}^k
v_i v_j k_m(x,x')
\right).
$$

Since we know that $(x^Tx')^m$ is a kernel, and by property 1 we know that all kernels are PSD, then this inner term

$$
\left(
\sum_{i=1}^k
\sum_{j=1}^k
v_i v_j k_m(x,x')
\right)
$$

must be greater than or equal to 0 for all m.

Notice that $\exp(x^Tx')$ is simply a weighted sum of these inner terms with positive weights $\frac{1}{m!}$. So, since a weighted sum of non-negative terms with positive weights must be non-negative, we know that $\exp(x^Tx')$ must be PSD. By theorem 2 we know that it must also be a kernel.

$\phi(x)$ for this kernel would be infinite dimensional. The first few terms would look something like this:

$$
\left[
1,
x_1,
x_2,
\ldots,
x_n,
x_1x_1\frac{1}{\sqrt2},
x_1x_2\frac{1}{\sqrt2},
\ldots,
x_nx_n\frac{1}{\sqrt2},
x_1x_1x_1\frac{1}{\sqrt3},
\ldots
\right].
$$

## 3. Attention with Kernels

### Slashing Runtime by O(n)

Now, recall our original issue. We want to be able to calculate

$$
a_i =
\frac{
\sum_{j=1}^n
\exp\left(\frac{q_i^T k_j}{\sqrt d}\right)v_j
}{
\sum_{j=1}^n
\exp\left(\frac{q_i^T k_j}{\sqrt d}\right)
}
$$

in constant time with respect to n.

Since we just showed that $\exp(x^Tx')$ is a kernel, let's rewrite this. I'm intentionally ignoring the $\sqrt d$ term to not clutter the equation, but in reality the kernel terms would be

$$
k\left(\frac{q_i}{\sqrt d},\frac{k_j}{\sqrt d}\right).
$$

Pluggin in the kernel

$$
a_i
=
\frac{
\sum_{j=1}^n k(q_i,k_j)v_j
}{
\sum_{j=1}^n k(q_i,k_j)
}
$$

and therefore

$$
a_i
=
\frac{
\sum_{j=1}^n
\langle\phi(q_i),\phi(k_j)\rangle v_j
}{
\sum_{j=1}^n
\langle\phi(q_i),\phi(k_j)\rangle
}.
$$

Plugging in the standard inner product

$$
\frac{
\sum_{j=1}^n
(\phi(q_i)^T\phi(k_j))v_j
}{
\sum_{j=1}^n
(\phi(q_i)^T\phi(k_j))
}.
$$

Note what this fraction is. The numerator is a weighted sum of the rows of V, where the weights are equal to a dot product. We are treating the rows of V as column vectors, meaning its dimensions are $(d_v,1)$.

If we instead treated the rows of V as row vectors, meaning giving it dimensions of $(1,d_v)$, we would get the same weighted sum of rows, just in row dimensions. Let's transpose the $v_j$ term.

$$
\frac{
\sum_{j=1}^n
(\phi(q_i)^T\phi(k_j))v_j^T
}{
\sum_{j=1}^n
(\phi(q_i)^T\phi(k_j))
}.
$$

Now notice that the $\phi(q_i)^T$ term can simply be pulled in front of the sum in both the numerator and denominator since it does not depend on j and its relationship with the other terms are simply multiplicative. Transposing the $v_j$ term was necessary to be able to do this without breaking our dimensions.

We are left with

$$
a_n = 
\frac{
\phi(q_n)^T
\sum_{j=1}^n
\phi(k_j)v_j^T
}{
\phi(q_n)^T
\sum_{j=1}^n
\phi(k_j)
}.
$$

Now imagine that we're generating a sequence of tokens using an LLM. After generating the $n-1$'th, we've calculated both

$$
\sum_{j=1}^{n-1}\phi(k_j)v_j^T
$$

and

$$
\sum_{j=1}^{n-1}\phi(k_j).
$$

To compute attention at the next step, we simply need to compute $\phi(q_n)$, $\phi(k_n)$, and $\phi(k_n)v_n^T$, all of which have runtimes independent of n. Now computing attention for an indiviual token when you have computed attention for the previous tokens is an $O(d)$ operation like we had hoped.

So, by using kernels and operating in the Hilbert space of $\phi$, we have eliminated the nonlinearities which bound the $q_i$ terms to the individual $k_i$ terms, thus enabling us to keep a running sum of the $k$ terms and the $kv$ terms, reducing the work by a factor of n.

### The Catch

There is a catch with this. While we have sliced the runtime by a factor of n, recall that the runtime of this operation is still $O(d)$, and our proof for why $\exp(x^Tx')$ is a kernel showed that the Hilbert space that this kernel operates in is one of infinite dimensions. So, our run time is now infinite.

So, we need to approximate this. One method I find particularly clever is one introduced by a paper titled *[Rethinking Attention with Performers](https://arxiv.org/abs/2009.14794)*.

Their key insight was that you could generate an unbiased estimate of $\exp(x^Tx')$ by creating a mapping $\phi$ with gaussian randomness. Specifically, they define

$$
\phi(x)
=
\frac{1}{\sqrt n}
\exp\left(
[
\omega_1^Tx-\frac{x^Tx}{2},
\omega_2^Tx-\frac{x^Tx}{2},
\ldots,
\omega_n^Tx-\frac{x^Tx}{2}
]
\right).
$$

Where each

$$
\omega_i\sim N(\mu=\boldsymbol 0,\Sigma=I).
$$

To see why this produces an unbiased estimate of $\exp(x^Tx')$, we can expand out a dot product between these two approximation mappings:

$$
\begin{aligned}
\phi(x)^T\phi(x')
&=
\frac{1}{n}
\sum_{i=1}^n
\exp\left(
\omega_i^Tx'
-\frac{x'^Tx'}{2}
+\omega_i^Tx
-\frac{x^Tx}{2}
\right) \\
&=
\frac{1}{n}
\sum_{i=1}^n
\exp\left(
\omega_i^T(x'+x)
-\frac{x'^Tx'}{2}
-\frac{x^Tx}{2}
\right) \\
&=
\frac{1}{n}
\sum_{i=1}^n
\exp(\omega_i^T(x'+x))
\exp\left(
-\frac{x'^Tx'}{2}
-\frac{x^Tx}{2}
\right).
\end{aligned}
$$

Taking expectation:

$$
E[\phi(x)^T\phi(x')]
=
\frac{1}{n}
\sum_{i=1}^d
E[\exp(\omega_i^T(x'+x))]
\exp\left(
-\frac{x'^Tx'}{2}
-\frac{x^Tx}{2}
\right).
$$

Notice that

$$
\omega^T(x+x')
=
\sum_{j=1}^d
\epsilon_j * (x_j + x'_j)
$$

where $\epsilon_j\sim N(0,1)$.

Multiplying a gaussian by a constant multiplies the variance of the gaussian by the constant squared. Summing independent samples from gaussians scale the variance additively. Thus,


$$
\omega^T(x + x') \sim N\left(0, \sum_{i=1}^d(x_i + x'_i)^2\right).
$$

Additionally,

$$
E[\exp(Z)] = \exp\left(\frac{\sigma^2}{2}\right)
$$

when $Z \sim N(0, \sigma^2)$.
Plugging back into our original equation:

$$
E[\phi(x)^T\phi(x')] =
\frac{1}{n}
\sum_{i=1}^n
\exp\left(
\frac{\sum_{i=1}^d(x_i + x'_i)^2}{2}
\right)
\exp\left(
-\frac{x'^Tx'}{2}
-\frac{x^Tx}{2}
\right).
$$

Note that:

$$
\sum_{i=1}^d(x_i + x'_i)^2
=
2x^Tx' + x^Tx + x'^Tx'.
$$

Plugging this back in we see:

$$
E[\phi(x)^T\phi(x')]
=
\frac{1}{n}
\sum_{i=1}^n
\exp(x^Tx')
=
\exp(x^Tx').
$$

So, in expectation this dot product will equal $e^{x^Tx'}$.

## 4. Relationship to Recurrences

If looked at correctly, linear attention is really just a recurrence. A recurrent system, when applied to sequences, uses a function of the i'th element and the i-1'th hidden state to generate the i'th hidden state, and a function to go from the ith hidden state to the ith output.

Specifically, it defines some function

$$
f(x_i, h_{i-1}) = h_i
$$

and some function

$$
g(x_i, h_i) = y_i,
$$

where $y_i$ is the output.

When we use linear attention, we essentially have two components to our hidden state which we need to keep track of. We need to keep track of

$$
\hat{h}_n
=
\sum_{j=1}^n
\phi(k_j)v_j^T
$$

and we need to keep track of

$$
\tilde{h}_n
=
\sum_{j=1}^n
\phi(k_j).
$$


We can update each component of our hidden state independently, with

$$
\hat{h}_n
=
\hat{h}_{n-1}
+
\phi(k_n)v_n^T
$$

and

$$
\tilde{h}_n
=
\tilde{h}_{n-1}
+
\phi(k_n).
$$

So, our recurrence function $f$ to compute the next hidden state is essentially

$$
f(x_n, \hat{h}_{n-1}, \tilde{h}_{n-1})
=
\left(
\hat{h}_{n-1}
+
\phi(x_n^TW_K)(x_n^TW_V)^T,
\;
\tilde{h}_{n-1}
+
\phi(x_n^TW_K)
\right).
$$

We can also define our function to generate the output

$$
g(x_n, \hat{h}_n, \tilde{h}_n)
=
\frac{
\phi(x_n^TW_Q)^T\hat{h}_n
}{
\phi(x_n^TW_Q)^T\tilde{h}_n
}.
$$

So, linear attention, and thus attention in general, is really just a recurrence with an infinite dimensional hidden state.


## Recap
That is it! Here is a recap of what was shown.
### 1. Attention
We first introduced attention and explained the optimization we're trying to make. For an LLM usecase, we want to be able to produce a sequence of length n in linear time. We are bottlenecked by this operation:

$$
a_i =
\frac{
\sum_{j=1}^i
\exp\left(\frac{q_i^T k_j}{\sqrt d}\right)v_j
}{
\sum_{j=1}^i
\exp\left(\frac{q_i^T k_j}{\sqrt d}\right)
}
$$

which is an O(n) operation required to calculate attention for a single token.

### 2. Kernels
Then we introduced the concept of a kernel. A kernel is a function 
$$
k: \chi \times \chi \to R
$$
that can be written as 
$$
\langle \phi(x), \phi(x') \rangle,
$$
for some mapping $\phi$. We then showed two important properties of kernels: that all kernels are symmetric PSD and that any symmetric PSD function is a kernel. We then used these properties to prove that three functions were kernels: dot products, polynomials of dot products, and exponentials dot products.

### 3. Linear Attention
Once equpped with knowledge of kernels, we finally introduced linear attention. The insight came from realizing that by replacing the $exp(q_i^Tk_j)$ term with a kernel formulation, we could pull the $q$ term out of the summation, and exploit the linearity in the kernel formulation by keeping a running sum of the previous $k$ and $kv$ terms. This meant that computing attention for an additional token after having done it for the previous tokens in that sequence was a constant time operation with respect to n.

The catch was that $\phi(x)$ is infinite dimensional for the exponential dot product kernel. So, we had to approximate it. We referenced one method of approximation, which does so by defining

$$
\phi(x) = \frac{1}{\sqrt{n}}exp([\omega_1^Tx - \frac{x^Tx}{2}], \omega_2^Tx - \frac{x^Tx}{2}], ... \omega_n^Tx - \frac{x^Tx}{2}])
$$

Where $\omega_i\sim N(\mu=\boldsymbol 0,\Sigma=I)$. We proved that in expectation, this finite mapping $\phi$ had the property that $E[\phi(x)^T\phi(x')] = exp(x^Tx')$.

### 4. Relationship to Recurrences
Finally, we showed an intersting connection between linear attention and recurrences, where we could view our running sum of $k$ and $kv$ terms in linear attention as two components of a hidden state in a recurrence. We showed that from this perspective linear attention, and attention in general, is simply a recurrence relation in infinte dimensions.

## Final Intuitions
If you don't remember all the math, the intuition is the following. We've made a non-linear function a linear one in a higher dimensional space. By doing this, we've replaced a non-linear bottleneck in attention with a linear operation, enabling us to compute $f(x_n)$ very easily from $f(x_{n-1})$. Since for the exact softmax operation used in attention the dimensionality of our vectors must be infinite, we use tricks to approximate the infinite dimensional linear operation in finite dimensions.

