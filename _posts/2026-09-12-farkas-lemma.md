---
layout: default
title: "Separability: Theory and Applications"
description: "This derives Farkas' Lemma and Steimke's Theorem using separability, leaning on as few unproven results as possible. It then discusses applications of the result which become largely trivial once the theory is well understood."
category: technical
image: /assets/images/farkas-lemma.png
---

# Separability: Theory and Applications

**Will Healy**
*September 2026*

## Introduction
I wanted to write a piece on separability because I think it's widely overlooked by practitioners of the fields which it touches. I'm refering particularly to the field of finance. Arbitrage pricing theory can be studied deeply without ever touching linear algebra. However, if you look at these same concepts through the lense of linear algebra, many results which originally seemed purely algebraic can be understood and visualized geometrically. I think this is quite cool.

## Separability
Separability refers to theorems and properties which make sets, points, subspaces, and objects separable, with an emphasis on linear separability. A linear separator is called a hyerplane, and is an n - 1 dimensional surface which divides an n-dimensional vector space into two halves. In $R^2$, a hyperplane is a line. In $R^3$, a hyperplane is a plane. For higher dimensions, these surfaces are hard to visualize but share the same properties. A halfspace refers to a portion of a vector space which lies on one side of a hyperplane. A hyperplane is typically defined by two values $c$ and $k$, where $c \in R^n$ and $k \in R$. It is defined as:

$$
\{x | c^Tx = k\}
$$

The set of points whose dot product with a certain vector equals a constant draws a surface perpindicular to the vector. The value k translates that surface along the vector, with $k = 0$ having the surface pass through the origin.

A halfspace is often defined as:
$$
\{x | c^Tx \leq k\}
$$
replacing $\leq$ with $<$, $>$, or $\geq$ also yield valid halfspaces.

### Convex Sets and Cones
A set is convex if it does not contain any two points such that the line segment connecting them contains a point no tin the set. For example a circle is a convex set. A star is not a convex set, since the points between two tips of the star lie outside of the star. Convex sets can be open or closed. A circle is a closed convex set. $R^n$ is an open convex set. A cone is type of open convex set defined as $\{Ax|x \geq 0\}$. It is literally shaped like a cone. Take the cone created by vectors [1, 2] and [1, 0]. The cone spanned by these two vectors looks like this:

![Cone spanned by (1,0) and (1,2)](/assets/images/cone-example.png)

### Separability of Disjoint Sets
Not all pairs of disjoint sets are strictly separable. This is not surprising. The set of all pairs of disjoint convex sets becomes closer, but still is not a sufficient condition. Take for instance the two disjoint and convex sets:

$$
C = \{(x, y) | y \geq e^x\}
$$

$$
D = \{(x, 0) | x \in R\}
$$

As $x \to -\infty$, $e^x \to 0$. Thus, these two sets become infinitely close and any hyperplane aiming to separate them would fail, since if the hyperplane sat any distance above the x-axis (which it needs to) it will eventually cross the line $e^x$.

#### Sufficient Condition for Separability
We need to add an additional condition for two disjoint sets to be strictly separable. The sufficeint condition for two convex sets to be separable at least one set is compact, and both sets are closed. A compact set is one where the set has finite bounds and contains its bounds. A closed set is one which contains its bounds. We will lean on this condition later on.

### Non-negative Orthant
The non-negative orthant is the simplest cone and is defined as

$$
\{x | x \geq 0\}
$$

In $R^2$ this is simply the first quadrant.
## Gordon Steimke Theorem

Say we have a matrix $A \in R^{m\times n}$. This matrix's image $Im(A)$ is defined as

$$
Im(A) = \{Ax | x \in R^n\}
$$

Now consider the question of whether there exists a vector $y \in R^m$ such that $y \neq 0$ and $y \in Im(A)$ and $y \geq 0$. That is, does this subspace overlap with the non-negative orthant anywhere but the origin. Let's assume that it does not. 

Then, there must exist $c > 0 \in R^m$ such that $A^Tc = 0$. That is, some element strictly inside the non-negative orthant must lie in the nullspace of $A^T$. To see why this is true, let's consider a separating hyperplane between the non-negative orthant and $C$. We are assuming that the only overlapping point between these two cones is at the origin. More rigourously

$$
Im(A) \cap R_+^m = \{0\}
$$

Since these two sets are not even disjoint, they are not separable. Let's instead consider the unit simplex defined as

$$
\Delta = \{x|x \geq 0, 1^Tx = 1\}
$$
Since this simplex lies within the non-negative orthant and does not include {0}, it must be disjoint from C. Additionally, this set is compact. Since both sets are convex, $Im(A)$ is closed, and $\Delta$ is compact, we know there must exist a separating hyperplane between these two sets. That is, there must exist some $c$ and $k$ such that

$$
z^Tc - k <> 0 \forall z \in Im(A)
$$

$$
\delta^Tc - k > 0 \forall \delta \in \Delta
$$

This is equivalent to writing

$$
\forall z \in Im(A): \forall \delta \in \Delta: z^Tc < \delta^Tc
$$

Now let's reason about what $c$ could be. $z$ must be orthogonal to every value in $Im(A)$. It's easiest to see why using contradiction. Imagine that for some $z \in Im(A)$ it was the case that $z^Tc \neq 0$. Since $Im(A)$ is a subspace and thus closed under scalar multiplication, we could define 

$$
\hat{z} = z * t
$$

where $t$ is a huge positive scalar if $z^Tc > 0$ and a huge negative scalar if $z^Tc < 0$. By scaling $t$ we could make $\hat{z}^Tc$ arbitrarily large, at some point surpassing $\delta^Tc$. This would break our separability requirement. So, we've reached a contradiction. For this reason, any separating hyperplane must be defined by some vector $c$ such that $A^Tc = 0$. In otherwords, c must lie in the null space of $A^T$. 

Now let's consider the elements of $c$. I claim that they must all be strictly positive.

Since ${0} \in Im(A)$, the separatiing hyperplane must be defined by a $k \geq 0$. Otherwise it would not be the case that $z^Tc - k < 0$ like is required for $z = {0}$. Additionally, since all the standard bases $e_1, e_2, ... e_m$ lie inside the unit simplex, it must be the case that $e_iTc - k > 0$, and since $k > 0$ this implies that $e_iTc = c_i > 0$ for $i = 1, 2, 3, ... m$. So $c > 0$.

This proves the following result:

$$
Im(A) \cap R_+^m = \{0\} \rightarrow \exists c \in R^m:  A^Tc = 0, c > 0
$$

This result, along with its converse which is also true, is called the Gordon Steimke Theorem. We will reference this result when looking at applications.

## Farkas' Lemma
Let's now imagine the cone defined by the columns of a matrix A

$$
C = \{Ax | x \geq 0\}
$$

Asking whether there exists some point $x \geq 0$ such that $Ax = b$ is equivalent to asking whether $b \in C$. If $b \notin C$, then since $C$ contains its bounds and $\{b\}$ is a compact set, by lemma 1 there must exist a strictly separating hyperplane separating $b$ from $C$. Additionally, since C is a cone, and thus spawns off at an acute angle from the origin, this strictly separating hyperplane can go through the origin, so long as we define the cone side of the sepeartion using an inclusive inequality. More simply, there must exist a separating hyperplane $c, k$ where

$$
\forall z \in C: z^Tc \geq k=0
$$

$$
b^Tc < k=0
$$

Since each $z$ can be written as $z = Ax$ for some $x \geq 0$, we can state:

$$
\forall x \geq 0: (Ax)^Tc = x^TA^Tc \geq k=0
$$

Since each $x$ can take on the values of any of the basis vectors $e_i$, this inequality can only hold if $A^Tc \geq 0$.

So, we have reached the conclusion that exactly one of the two following results are true:

$$
\exists x \geq 0: Ax = b
$$

OR

$$
\exists c: A^Tc \geq 0, b^Tc < 0
$$

## Applications
Now we will aim to apply these results to two fields: finance and optimization

### Risk Neutral Pricing
Say that we have a payout matrix of assets. Say that this is a matrix 

$$
A \in R^{m \times n}
$$

where m is the number of states which could exist in the next time period and n is the number of assets in the universe. We will define $A_{ij}$ to be the price of asset j in state i.

You can construct a portfolio $x \in R^n$ where $x_j$ represents the quantity of asset j which you posess. $x_i$ can be positive or negative. Investing in a portfolio $x$ costs $p^Tx$ dollars. If $p^Tx$ is negative, then this means you are being paid to take on this portfolio, and if it is positive it means that you are payiing to take on this portfolio. The value of portfolio $x$ in state i is equal to $a_i^Tx$, where $a_i$ is the i'th row of A. If for a portfolio x

$$
Ax \geq  0
$$

Then, this portfolio is guaranteed to have a non-negative after one time step. If both

$$
Ax \geq 0
$$

$$
p^Tx < 0
$$

then this is an arbitrage opportunity, since we're getting paid to take on this portfolio and we can't possibly owe money at the end of the time step.
Let's let 

$$
B = \begin{bmatrix} -p^T \\ A \end{bmatrix} \in R^{(m+1) \times n}
$$ 

as a column vector. Then, our arbitrage opportunity exists when

$$
Bx = \begin{bmatrix} -p^Tx \\ Ax \end{bmatrix} \geq 0
$$

This is eqivalent to saying that there is an x such that a$Bx$ is in the non-negative orthant. We exclude from the non-negative orthant since this is trivially satisfied with $x=0$ and would not make any money.

$$
\exists x: Bx \in R_+^{m+1} \backslash \{0\}
$$

Now recall our proof of the Gordon Steimke Theorem. We proved that

$$
Im(A) \cap R_+^m = \{0\} \rightarrow \exists c \in R^m:  A^Tc = 0, c > 0
$$

Replacing A with B, we see that our arbitrage condition failing implies that there exists a $c$ such that

$$
B^Tc = 0, c > 0
$$

Let's let $c = [a, \tilde{q}]$ where $a \in R$ and $q \in R^m$ Expanding $B^Tc$

$$
B^Tc = [-p, A^T]c = -pa + A^T\tilde{q} = 0
$$

$$
pa = A^T\tilde{q}
$$

Now let us define $q = \frac{\tilde{q}}{a}

$$
p = A^Tq
$$

where we still know that $q > 0$.

Now imagine any reasonable asset matrix. One option you can do with your money is put it under your mattress. It will be there unchanged in the next time step. So, it is only reasonable for there to be some asset which has a payout equal to its cost in every state. That is, some asset j such that $p_j = 1$ and the j'th column of A is a vector of 1s. Looking at the result we just derived, this would mean that for this risk free asset, we have

$$
p_j = 1 = \sum_{i=1}^mq_i
$$

This means that the values of $q$ are all positive and sum to 1, making $q$ a probability distribution. More rigorously, if $q_i$ represents the probability of ending up in state i, then the assets are all priced risk neutrally. So, we have shown the following set of strong alternatives in finance:

Either there is an arbitrage opportunity

OR

The assets are priced risk neutrally according to some probability distribution q.




### Duality and Linear Programming (coming soon)
