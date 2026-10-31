---
layout: default
title: "Set Regressors Theory Deep Dive"
description: "This does a deep dive into the theory behind set regressors, specifically citing the [Deep Sets](https://arxiv.org/pdf/1703.06114) paper. It first introduces the broad concepts of set regressors, rigorously proves a theorem which makes them possible, and then introduces the technices and results behind the paper."
category: technical
image: /assets/images/set-regressors/thumbnail.jpg
---

# Set Regressors Theory Deep Dive

**Will Healy**
*September 2026*

## Introduction
Set regressors are an interesting concept. Philosophically, they are very simple. They are learnable functions which take a set as an input and output a continuous value. But if you consider the methods which are used to regress a continuous value against a traditional input, i.e a vector, you'll notice that they rely on the input having both a consistent ordering and fixed number of features. Sets have neither of these properties. It's interesting to try, and fail, to visualize the set of all subsets of some universe. They don't live in a nice visualizable space the way vectors in a vector space do. Thus, the techniques to map this domain onto a continuous range in an intelligent way must adapt. The way traditional learning techniques can be adapted to this foreign domain is shockingly clean.

Set regressions pop up in several corners of machine learning. Typically they are used as a small component in a larger system, though, and thus the theory behind them is easily overlooked. This is why I wanted to write this: to prove rigorously why these techniques are theoretically sound and also explain intuitively how they can be so effective despite the disorderedness of their domain.

I was first introduced to these ideas in a class taught by Jure Leskovec, a prominent ML researches whose work focuses on machine learning with graphs. Set regressors appear in graph ML when embedding a node in a graph. The learned embedding of a node typically comes from several iterations of embedding features of the node of interest, aggregating this embedding with the embeddings of neighboring nodes, and repeating this process to get a positionally aware embedding of a node.

I've also used these techniques in industry. My application came from needing to look at data from a classification model and determine how much more data was necssary to have certain guarantees over our performance. The classification models could be for any number of classes, and thus the model we learned needed to take as input class distributions from data with a varying numbers of classes. This is simply a set. We tried several methods for doing this. Most methods did not involve using set regressor techniques and instead derived metrics from the distribution and jammed them into a fixed length vector. However, the set regressor worked the best.

These techniques are very powerful. The mathematical properties which power them are also very aesthetic. The [Deep Sets](https://arxiv.org/pdf/1703.06114) paper is one of my favorite papers ever. It takes a very non-trivial mathematical result, proves it rigorously leaning on concepts from abstract algebra, set theory, and even topology, and then introduces an architecture based on the result which has immense practical value. It's fascinating.


## Prerequisites

### Permutation Invariance

A function from $R^n \to R^m$ is permutation invariant if rearranging the order of the inputs does not change the output. That is, $f([1, 2]) = f([2, 1])$. More generally, a function $f$ is permutation invariant if:

$$
f(Px) = f(x)
$$

where $P$ is a permutation matrix. A permutation matrix is simply an $n \times n$ matrix where each row and each column have exactly one non-zero element which is a 1. 

### Permutation Equivariance

A function from $R^n \to R^m$ is permutation equivariant if rearranging the order of the inputs changes the output in a *predictable* way. A predictable way simply means that for every permutation matrix $P \in R^{n \times n}$ there is a corresponding permutation matrix $Q_P \in R^{m \times m}$ such that:

$$
f(Px) = Q_Pf(x)
$$

Permutation equivariance is a looser requirement than invariance. That is, all permutation invariant functions are permutation equivariant functions where $Q_P = I$.

For functions with a one dimensional output, meaning $m = 1$, permutation equivariance and permutation invariance is the same thing. 

*Please note that there is a broader group theory definition of equivariance which differs from this interpretation. In this group theory definition, $Q_P$ does not need to be a permutation matrix, but simply needs to satisfy the condition that $Q_{P_1P_2} = Q_{P_1}Q_{P_2}$. A function with a 1-dimensional output can be permutation equivariant and not permutation invariant under this definition.*

## Intuitions on Set Regressors

Set regressors learn a permutation invariant function of the elements in the set. Even though sets are not ordered, they still must be placed in an array on a computer. Since we're trying to learn a mapping from a set to an output, two orderings of the same set must be fed through a function which is permutation invariant. Otherwise, the same set could be mapped to two different outputs.

### A First Attempt at Permutation Invariance

When I first heard about this it sounded silly to me. I wondered why not just define an arbitrary ordering to the elements of the set based on some rule, such as by sorting, and then apply regular learning techniques to this vector. Specifically, if you learn a function $f$ over the sorted set, then your final function would be $g$ defined as:

$$
g(x) = f(s(x))
$$

Where $s(x)$ sorts the elements of x. This would guarantee that the same set gets mapped to the same output because it will first get mapped to the same ordering. This makes an especially large amount of sense if you're regressing over sets of fixed cardinality. I think it's funny to imagine being given data which is pre-sorted. For example say you were given a list of sorted heights of a group of kids and were told to predict what grade they were in. Your first thought would never be to first scramble the elements and then work with this unordered object. The sorted object contains at least as much information as the unordered object. Yet, here we are.

There's a qwirk with the sorting method. The issue doesn't exist for sets of numbers. But for sets of vectors, sorting the elements and then learning a continuous function of this sorted list can learn discontinuous functions of the original object.

For example imagine that you're sorting vectors in $R^2$ by considering the value of their first element, using each subsequent element to break ties. Take the two vectors $x_1 = [0, 0]$ and $x_2 = [\epsilon, 1]$. If the learned function is:

$$
f(y_{11}, y_{12}, y_{21}, y_{22}) = y_{22},
$$

then epsilon going from $\epsilon = 0.01$ to $\epsilon = -0.01$ would change the value of this function from 1 to 0, since the ordering $s(x_1, x_2)$ would go from $[x_1, x_2]$ to $[x_2, x_1]$. Thus, $y_{22}$ would go from being 1 to being 0.

This may be what you want in specific scenarios, but typically discontinuous functions are not good in regressions.


### Deep Sets Approach

If we're not going to use the sort then learn approach, another way we could guarantee permutation invariance is to structure our learned function as:

$$
f(x) = \rho(\sum_i \phi(x_i))
$$

This exploits the fact that summing elements is a permutation invariant operation. Since multiplying elements is a permutation invariant function as well, another canidate with the same property would be to use:

$$
f(x) = \rho(\prod_i \phi(x_i)),
$$

where the product of two vectors is their elementwise product.

The first approach is the method used by the deep sets architecture. As we will show below, $\rho$ and $\phi$ can be chosen to express every single permutation invariant function for countable universes, and every single continuous permutation invariant function for uncountable universes. This result is the primary contribution of the deep sets paper.


## Primary Result of Deep Sets

Any continuous permutation invariant function $f$ can be written as:

$$
f(x) = \rho(\sum_i \phi(x_i))
$$

When I first read this I was very surprised. This is not at all obvious. I started trying to think of counter examples. I first tried the permutation invariant function:

$$
f(x) = \prod_i x_i
$$

This can be written in the required format by letting $\phi(x) = ln(x)$ and letting $\rho(x) = e^x$. Then, we see that:

$$
f(x) = exp(\sum_i(ln(x_i))) = \prod_i exp(ln(x_i)) = \prod_i x_i
$$

So, it can be done.

We will now prove the existence of a $\phi$ and $\rho$ for all permutation invariant functions. This will be a bit of an adventure. We will divide this up into several scenarios which cover both countable and non-countable cases.

### Countable Case

A set that is countable is one who's values can all be assigned a natural number. All finite sets are countable. Some infinite sets are countable. The set of natural numbers is obviously a countable set since each element can be assigned to itself. The set of all real numbers is not countable, since for every natural number there are infinitely many real numbers.


#### Finite Countable Case

Imagine a finite universe of elements $U$. Since the universe is finite, and thus countable, we can give an arbitratry assignment of each element to a natural number. Let's let $N(u)$ be the natural number we assign to the element $u \in U$. To give an example of this we can consider the universe of elements: {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}. Since each element is already a natural number we can assign each number to itself. So, $N(u) = u$.

If $f$ is a function over this universe, then for each $X \in \mathcal{P}(U)$, a function $f$ must map $X$ to an associated value $f(X)$. Here $\mathcal{P}(U)$ is the power set of $U$ and is defined as:

$$
\mathcal{P}(U) = \{X | X \subseteq U \}
$$

We want to show that $f$ can be expressed as:

$$
\forall X \in \mathcal{P}(U): f(X) = \rho (\sum_i \phi(x_i))
$$

Let's think about the requirements of $\phi$ and $\rho$ to make that happen. If $f$ is injective, then in order for this to work it needs to be the case that $(\sum_i \phi(x_i))$ is also injective. That is, by summing the $\phi(x_i)$ terms we don't lose information. If this was not the case, then for some two subsets $X_1$ and $X_2$ it would be the caset that:

$$
\sum_{x_{1i} \in |X_1}\phi(x_{1i}) = \sum_{x_{2i} \in |X_2}\phi(x_{2i})
$$

and thus:

$$
\rho(\sum_{x_{1i} \in |X_1}\phi(x_{1i})) = \rho(\sum_{x_{2i} \in |X_2}\phi(x_{2i}))
$$

But since we know $f$ is injective this must be wrong. So, $(\sum_i \phi(x_i))$ must be injective. If $(\sum_i \phi(x_i))$ is injective, since the universe is finite we can simply define $\rho$ to be a mapping from each of these sums to it's corresponding value $f(X)$.

In the finite case, we can achieve an injective $(\sum_i \phi(x_i))$ aggregation of the elements by defining $\phi(x) = 2^{-N(x)}$. For example if we consider the set {1, 4, 5}, we would have that:

$$
\sum_i \phi(x_i) = \sum_i 2^{-N(x_i)} = \frac{1}{2} + \frac{1}{16} + \frac{1}{32}
$$

Then, $\rho$ could simply be a mapping which maps 

$$
(\frac{1}{2} + \frac{1}{16} + \frac{1}{32}) \to f(\{1, 4, 5\})
$$

Note that we're not trying to say what $\rho$ would be for an arbitrary function $f$. We're instead showing that some mapping must exist. The only condition for this is an injective mapping from $X$ to $\sum_i \phi(x_i)$. So long as this is injective, meaning that each input maps to a unique output, then this sum can be later mapped to $f(X)$.

#### Infinite Countable Sets

In the infinite case, we run into a subtle issue with our choice of $\phi$. The issue arises from the fact that:

$$
\sum_{i=1}^{\inf} 2^{-i} = 1
$$

And thus:

$$
\sum_{i=2}^{\inf} 2^{-i} = 0.5 = \phi(1)
$$

So, for the sets 

$$
X_1 = \{1 \}
$$ 

and 

$$
X_2 = \mathbb{N}  \setminus \{ 1 \} = \{2, 3, 4, 5, .....\}
$$

We have a collision since:

$$
\sum_{x_{1i} \in X_1}\phi(x_{1i}) = \sum_{x_{2i} \in X_2} \phi(x_{2i})
$$

So, it's not quite lossless. Luckily, any number greater than 2 does not suffer from these collisions. The Deep Set paper displays this with the choice of 4. That is, it defines:

$$
\phi(x) = 4^{-N(x)}
$$

However, any number greater than 2 will work. 

We've just shown that the primary result holds for countable universes.

### Non-countable Universes

For non-countable universes the problem becomes less trivial. As with the countable case, the biggest challenge is creating an injective mapping of $X$ of the form:

$$
\sum_i \phi(x_i)
$$

When we can't assign each $x_i$ to a natural number, our trick of defining:

$$
\phi(x) = 4^{-N(x)}
$$

doesn't work. To show that this is still possible we'll rely on the following result.

For any set of real numbers $S$, it's the case that 

$$
[ \sum_{x_i \in S} x_i^0, \sum_{x_i \in S} x_i^1, \sum_{x_i \in S} x_i^2, \sum_{x_i \in S} x_i^3, ... \sum_{x_i \in S} x_i^{|S|} ]
$$

is unique to $S$. That is, if we let 

$$
\phi(x) = [1, x, x^2, ... x^{|M|}]
$$

Then we can define our sum $E(X)$ as:

$$
E(x) = \sum_i \phi(x_i) = [ \sum_{x_i \in S} x_i^0, \sum_{x_i \in S} x_i^1, \sum_{x_i \in S} x_i^2, \sum_{x_i \in S} x_i^3, ... \sum_{x_i \in S} x_i^{|S|} ]
$$

which is an injective function of $X$. If we define $\rho$ to simply be a mapping from $E(x) \to f(x)$, we have proven the existence of a function $\rho$ and a function $\phi$ such that:

$$
f(x) = \rho(\sum_i \phi(x))
$$

To make this more concrete, note that any injective function $f$ has an inverse $f^{-1}$. This falls out directly by the definition of injectivity. Thus, if $E(x)$ is injective, then there is a function $E^{-1}$ such that $E^{-1}(E(x)) = X$.

If we let 

$$
\rho(x) = f(E^{-1}(x)),
$$

then it's the case that:

$$
\rho(\sum_i \phi(x_i)) = \rho(E(x)) = f(E^{-1}(E(x))) = f(x)
$$

So, we would have recovered our function $f$.

### Proof of the Non-countable Case

We are going to prove that for:

$$
\phi(x) = [1, x, x^2, ... x^{|M|}]
$$

it is the case that

$$
E(X) = \sum_i \phi(x_i) = 

[ \sum_{x_i \in S} x_i^0, \sum_{x_i \in S} x_i^1, \sum_{x_i \in S} x_i^2, \sum_{x_i \in S} x_i^3, ... \sum_{x_i \in S} x_i^{|S|} ]
$$

is an injective function of $X$. This is a beautiful proof which covers a wide breadth of fields.

#### A small example to build intuition

We will start with the case where $\|X\| = 2$.

Let $$X = \{x_1, x_2\} $$ be a set in $\mathcal{P}(U)$. Then:

$$
\begin{aligned}
\phi(x) &= [x, x^2] \\

\sum_i^2 \phi(x_i) &= [x_1 + x_2, x_1^2 + x_2^2] = [p_1, p_2]
\end{aligned}
$$


We must show that from $p_1$ and $p_2$ we can recover $x_1$ and $x_2$. To do this we are going to use properties of polynomials. Specifically, we are going to use the fact that a polynomial is uniquely defined by its roots and their associated multiplicities. This claim comes from a small extension of the fundamental theorem of algebra, a result beyond the scope of this essay.

Let's go slowly to build intuition. For a given set of cardinality 2, the polynomial we are going to think about is:

$$
\begin{aligned}
\prod_{i=1}^2(t-x_i) &= (t-x_1)(t-x_2) \\
&= t^2 - tx_1 - tx_2 + x_1x_2
\end{aligned}
$$

Since this polynomial is defined by roots $x_1$ and $x_2$, if we can define this polynomial in terms of $p_1$ and $p_2$, then we've proven injectivity of $\sum_i\phi(x_i)$. Make sure this is well understood as the whole proof is around showing this. To be clear the reasoning for this is:

1. A polynomial is uniquely defined by its roots and associated multiplicites.
2. We define a polynomial with roots equal to the elements of the set $$ \{x_1, x_2, ... x_n \}$$.
3. We define values $p_j = \sum_i^n x_i^j$ for $j$ from 1 to n.
4. If we can formulate the polynomial whose roots are the $x_i$ terms from only the $p_j$ terms, then we've lost no information by bullet point 1.

Back to our proof, we want to show that we can write:

$$
g(t) = t^2 - tx_1 - tx_2 + x_1x_2
$$

using

$$
\begin{aligned}
p_1 = x_1 + x_2 \\

p_2 = x_1^2 + x_2^2
\end{aligned}
$$

The $-tx_1 - tx_2$ term is simple to write in terms of $p$. It's just:

$$
-tx_1 - tx_2 = -tp_1
$$

To see how we can write the $x_1x_2$ term in terms of $p$, we can first consider the term $p_1^2$:

$$
\begin{aligned}
p_1^2 &= (x_1 + x_2)^2 \\
&= x_1^2 + x_2^2 + 2x_1x_2 \\
&= p_2 + 2x_1x_2
\end{aligned}
$$

And thus:

$$
x_1x_2 = \frac{p_1^2 - p_2}{2}
$$

This gives us our final result that a polynomial $g(t)$ with roots $x_1$ and $x_2$, each of which multiplicy 1 can be written as:


$$
g(t) = t^2 - tp_1 + \frac{p_1^2 - p_2}{2}
$$

where

$$
\begin{aligned}
p_1 = x_1 + x_2 \\
p_2 = x_1^2 + x_2^2
\end{aligned}
$$

**Thus, for the $ \|X\| = 2 $ case we know that $E(X) = \sum_i^2 \phi(x_i)$ is injective.**

#### Proof for arbitrary $|X|$.

We will now prove this for arbitrary sized sets $X$. This is the heart of the mathematical result proved by the [Deep Sets](https://arxiv.org/pdf/1703.06114) paper. Let's first understand a concept called Newton's identities.

We will take the same approach with this proof. That is, we are trying to show that:

$$
g(t) = \prod_i^{|X|}(t - x_i)
$$

can be written in terms of $ p_1, p_2, ... p_{\|X\|} $. To do this we will first consider the derivative of the logarithm of $g(t)$. The derivative of the logarithm of a function $g$ is equal to the derivative of $g$ divided by $g$. That is, $\frac{g'(t)}{g(t)}$.

Recall that by the product rule if you're differentiating a product of elements, you can take the derivative of each element and multiply it by the remaining elements, then sum this result over all elements to get the derivative of the product. So, since the derivative of $t - x_j$ with respect to $t$ is just 1, the derivative of this product is simply the sum over all products of unique $\|X\| - 1$ elements:

$$
g'(t) = \sum_i^{|X|}\prod_{j\neq i}^{|X|} (t - x_j)
$$

to get our desired term $\frac{g'(t)}{g(t)}$, we can divide both sides by 

$$
g(t) =  \prod_{j}^{|X|} (t - x_j)
$$

which yields:

$$
\begin{aligned}
\frac{g'(t)}{g(t)} &= \frac{\sum_i^{|X|}\prod_{j\neq i}^{|X|} (t - x_j)}{\prod_{j}^{|X|} (t - x_j)} \\
&= \sum_i^{|X|} \frac{1}{t - x_i}
\end{aligned}
$$

We're now going to apply a clever trick which exploits properties of geometric series. If we factor out the term $\frac{1}{t}$ from this we can write:

$$
\frac{g'(t)}{g(t)} = \frac{1}{t} \sum_i^{|X|} \frac{1}{1 - \frac{x_i}{t}}
$$

We're now going to invoke another property of polynomials, which is that if you observe any n + 1 points of an n-degree polynomial, these values uniquely define the polynomal. Thus if you observe infinite points of any polynomial this will uniquely define the polynomial. Since there are infinite points of the polynomial $g(t)$ where $t > max_i(\|x_i\|)$, we can concern ourself only with the region of the polynomial where $t > max_i(\|x_i\|)$ without losing information about the polynomial.

In this region where $t > max_i(\|x_i\|)$, it's the case that the term $\frac{x_i}{t}$ has an absolute value less than 1.

Now note that a geometric series over $|z| < 1$ it's the case that:

$$
\sum_{i=0}^{\infty} z^i = \frac{1}{1-z}
$$

Thus by considering the region where $t > max_i(\|x_i\|)$ we have it that:

$$
\frac{1}{1 - \frac{x_i}{t}} = 1 + \frac{x_i}{t} + \frac{x_i^2}{t^2} + \frac{x_i^3}{t^3} + ... \infty
$$

And thus:

$$
\frac{1}{t-x_i} = \frac{1}{t} (\frac{1}{1 - \frac{x_i}{t}}) = \frac{1}{t} + \frac{x_i}{t^2} + \frac{x_i^2}{t^3} + \frac{x_i^3}{t^4} + ... \infty
$$

Equipped with this new formulation of $\frac{1}{t-x_i}$, we can now write:

$$
\begin{aligned}
\frac{g'(t)}{g(t)} &= \sum_{i=1}^{|X|} \frac{1}{t-x_i} \\
&= \sum_{i=1}^{|X|}\frac{1}{t} (\frac{1}{1 - \frac{x_i}{t}}) \\
&= \sum_{i=1}^{|X|}\frac{1}{t} + \frac{x_i}{t^2} + \frac{x_i^2}{t^3} + \frac{x_i^3}{t^4} + ... \infty \\
&= \frac{|X|}{t} + \frac{p_1}{t^2} + \frac{p_2}{t^3} + \frac{p_3}{t^4} + ... \infty
\end{aligned}
$$

And thus:

$$
g'(t) = g(t)(\frac{|X|}{t} + \frac{p_1}{t^2} + \frac{p_2}{t^3} + \frac{p_3}{t^4} + ... \infty)
$$


When $t > max_i(x_i)$.

Let's stop here and think about what this means...

Now we will consider another formulation of $g'(t)$ . Since $g$ is a polynomial of degree $\|X\|$, we can write $g$ as:

$$
g(t) = t^{|X|} + e_1 t^{|X|-1} + e_2t^{|X|-2} + ... e_{|X|}
$$

and $g'$ as:

$$
g'(t) = (|X|)t^{|X| - 1} + (|X|-1)e_1 t^{|X|-2} + (|X| - 2)e_2t^{|X|-3} + ... e_{|X|-1}
$$

Now recall our original formulation of $g'(t)$:

$$
g'(t) = g(t)(\frac{|X|}{t} + \frac{p_1}{t^2} + \frac{p_2}{t^3} + \frac{p_3}{t^4} + ... \infty)
$$

By setting both formulations of $g'$ equal to each other we see that:

$$
\begin{aligned}
g'(t) &= g(t)(\frac{|X|}{t} + \frac{p_1}{t^2} + \frac{p_2}{t^3} + \frac{p_3}{t^4} + ... \infty) \\
&= (t^{|X|} + e_1 t^{|X|-1} + e_2t^{|X|-2} + ... e_{|X|})(\frac{|X|}{t} + \frac{p_1}{t^2} + \frac{p_2}{t^3} + \frac{p_3}{t^4} + ... \infty) \\
&= (|X|)t^{|X| - 1} + (|X|-1)e_1 t^{|X|-2} + (|X| - 2)e_2t^{|X|-3} + ... e_{|X|-1}
\end{aligned}
$$

Now let's look at the last line. If we can recover the $e_i$ terms using only $p$ terms then we have not lost any information. 

Let's think about the $e_1$ term in the third line. In the derivative, this is associated with the term:

$$
(|X|-1)e_1t^{|X|-2}
$$

Looking at the product in line 2, the only way which we could get a term with $t^{\|X\|-2}$ is by multiplying the $t^{\|X\|}$ term with the $\frac{p_1}{t^2}$ term or by multiplying the $e_1t^{\|X\| - 1}$ term with the $\frac{\|X\|}{t}$ term. Thus, we can write:

$$
(|X|-1)e_1 = p_1 + e_1|X|
$$

which yields:

$$
e_1 = -p_1
$$

Now let's think about the $e_2$ term:

$$
(|X| - 2)e_2t^{|X|-3}
$$

The only way that we could get 

$$
t^{|X|-3}
$$

terms by the product on line 2 are by:

$$
\begin{align}
t^{|X|}\frac{p_2}{t^3} \\
e_1t^{|X|-1}\frac{p_1}{t_2} \\
e_2t^{|X|-2}\frac{|X|}{t}
\end{align}
$$

Thus, the value for $(\|X\| - 2)e_2t^{\|X\|-3}$ must be the sum of these:

$$
(|X| - 2)e_2t^{|X|-3} = t^{|X|}\frac{p_2}{t^3} + e_1t^{|X|-1}\frac{p_1}{t_2} + e_2t^{|X|-2}\frac{|X|}{t}
$$

dividing everything by $t^{\|X\|-3}$:

$$
(|X| - 2)e_2 = p_2 + e_1p_1 + e_2|X|
$$

And so:

$$
e_2 = \frac{p_2 + e_1p_1}{-2}
$$

This pattern trivially continues. That is, you can write the $e_i$ term in terms of the $e_{j< i}$ terms and the $p_j$ terms.


For an arbitrary $e_k$ term: $(\|X\| - k) e_kt^{\|X\| - k - 1}$, we can see:

$$
(|X| - k) e_k = \sum_{i=0}^k p_ie_{k-i}
$$

And since we know $p_0 = \|X\|$ we can reformulate this as:

$$
|X|e_k - ke_k= |X|e_k + \sum_{i=1}^k p_ie_{k-i}
$$

Subtracting $\|X\|e_k$ both sides:

$$
- ke_k = \sum_{i=1}^k p_ie_{k-i}
$$

and thus:

$$
e_k = \frac{1}{-k}\sum_{i=1}^k p_ie_{k-i}
$$

So, since we can recover the coefficients $(e_1, e_2, e_3... e_{\|X\|})$ of the polynomial $g(t)$ whose roots are the elements of $X$, we have shown that our mapping from

$$
X \to [p_1, p_2, ... p_{|X|}]
$$

is injective. So we are done! To summarize this proof, we have done the following. We have proposed a candidate function $\phi(x) = [x, x^2, x_3, ... x_{\|X\|}]$ such that:

$$
E(X) = \sum_{i=1}^{|X|}\phi(x_i)
$$

is injective. One caveat here is that $\phi$ must know $\|X\|$, and thus this only works when the number of elements in the set is fixed.