---
layout: default
title: "Separability and Duality in Linear Programs"
description: "This derives Farkas' Lemma using separability, leaning on as few unproven results as possible. It then uses this result to understand duality in linear programming."
category: technical
image: /assets/images/farkas-lemma.png
---

# Separability and Duality in Linear Programs

**Will Healy**
*September 2026*

## Introduction
The goal of this piece is to understand duality in linear programming. To do this, we will first build strong intuitions on the separability of cones and points using Farkas' Lemma. Once separability is well understood, we will apply this concept to linear programming, leading to both a motivation and derivation of the dual problem for a linear program. This piece assumes you have knowledge of hyperplanes, halfspaces, convext sets, and cones. The exact required prerequisite information for hyperplanese and halfspaces can be found [here](/notes/separability/) and the required preerquisite knowledge for convext sets and cones can be found [here](/notes/convex-sets/).



## Farkas' Lemma
Let's now introduce Farkas' lemma. Imagine the cone $C$ defined by the columns of a matrix A:

$$
C = \{Ax | x \geq 0\}
$$

Asking whether there exists some point $x \geq 0$ such that $Ax = b$ is equivalent to asking whether $b \in C$. If $b \notin C$, then since both $C$ and $b$ are convex sets, $C$ contains its bounds, and $\{b\}$ is a compact set, there must exist a strictly separating hyperplane separating $b$ from $C$. For intuition on why these criteria make sense please see [here](/notes/separability/).

Additionally, since C is a cone, and thus spawns off from the origin, this strictly separating hyperplane can go through the origin. To see why this is true, take any cone and a point outside the cone, then define a strictly separating hyperplane. We will define hyperplanes with two values $c$ and $k$ as $$\{x \\| c^Tx = k \}$$. Thus a separating hyperplane between our cone $C$ and point $b$ gives us:

$$
\forall z \in C: z^Tc \geq k
$$

$$
b^Tc < k
$$

If $$k$$ is non-negative, then k must equal $0$ because $$\{0\} \in C$$. If $$k$$ is negative, then either there exists some $$z$$ such that $$z^Tc < 0$$ and the negative $$k$$ is necessary to truly separate these sets, or ther while still being a separating hyperplane. If there did exist a $$z$$ with this property of $$z^Tc < 0$$, then since cones are closed under positive scalar multiplication, we could define $$z' = tz \in C$$ for an arbitrary large positive $$t$$. With large enough $t$, $z'^Tc$ could become arbitrarily large in the negative direction and eventually become less than $$k$$. Thus, there must not exist a $$z$$ such that $$z^Tc < 0$$. This means any valid hyperplane orthogonal to $c$ that separates a point from a cone can be translated to go through the origin without losing its separating properties.


Algebraically, this means that there must exist a separating hyperplane $c, k$ where

$$
\forall z \in C: z^Tc \geq k=0
$$

$$
b^Tc < k=0
$$

Since each $z$ can be written as $z = Ax$ for some $x \geq 0$, we can state:

$$
\forall x \geq 0: (Ax)^Tc = x^TA^Tc \geq 0
$$

Now note that each $x$ can take on the values of any of the basis vectors $e_i$, so this inequality can only hold if $A^Tc \geq 0$.

So, we have reached the conclusion that exactly one of the two following results are true for a matrix $A$ and a point $b$: 

**Either $b$ is in the cone of $A$'s columns or there is a strictly separating hyperplane separating passing through the origin which separates $b$ and $A$.** 

In math, the following two statements are strong alternatives for any matrix A and a vector b:

$$
\exists x \geq 0: Ax = b
$$

OR

$$
\exists c: A^Tc \geq 0, b^Tc < 0
$$


## Duality and Linear Programming

This section will do three things: it will introduce the concept of linear programming, define duality, and it will show how Farkas' Lemma is a useful tool for reasoning about these concepts.

### Linear Programming
A linear program can be defined several ways. I was taught linear programming by Stephen Boyd. As a result, the formulation he uses in his book [Convex Optimization](https://web.stanford.edu/~boyd/cvxbook/bv_cvxbook.pdf) is the one which feels most natural to me. He defines a linear program as an optimization problem of the form:

$$
min_x c^Tx
$$

Subject to

$$
Ax \leq b
$$

I will call this formulation form 1. Notice what this is asking. This is asking us to find a vector x which minimizes the dot product between x and some constant vector c, constrained by the fact that x must lie in an intersection of halfspaces. Each halfspace is defined by:

$$
a_i^Tx \leq b_i
$$

where $a_i$ is the i'th row of A.

A different formulation of a linear program, which I will call form 2, is:

$$
max_x c^Tx
$$

Subject to 

$$
Ax = b
$$

$$
x \geq 0
$$

These two formulations are equivalent in expressability. This means that any optimization problem of the first form has an equivalent optimization problem in the second form. Showing why is a slight tangent to the main ideas of this piece, but I think it's a useful exercise, especially when trying to think about linear programs geometrically. We will show one direction of this equivalence.

### Proof of Equivalence

Let's assume we have an optimization problem in the first form. That is, we have some problem:

$$
min_x c^Tx
$$

Subject to

$$
Ax \leq b
$$

We will show that this can be written in the second form.

Let's first introduce slack variables. Specifically, if $Ax \leq b$, then it must be the case that $Ax + s = b$ for some non-negative vector s. So, we could define the problem as:

$$
min_{\{x, s\}}c^Tx
$$

subject to

$$
Ax + s = b
$$

$$
s \geq 0
$$


Next we can define an equivalent problem by defining:

$$
x = x_+ - x_-
$$

where

$$
x_+ > 0
$$

$$
x_- > 0
$$

And so this can be written as:

$$
min_{\{x_+, x_-, s\}} c^T(x_+ - x_-) = c'^Tx'
$$

where

$$
c' = \begin{bmatrix} c \\ -c \end{bmatrix}
$$

and

$$
x' = \begin{bmatrix} x_+ \\ x_- \end{bmatrix}
$$

Subject to:

$$
\begin{align}
Ax_+ - Ax_- + s = b \\
x_+ \geq 0 \\
x_- \geq 0
\end{align}
$$

Which is equivalent to:

$$
\begin{align}
A'x' + s = b \\
x' \geq 0 \\
\end{align}

$$

where $A'$ is defined as $[A, -A]$

Finally, we can place s into the same vector we're optimizing over and define our problem as:

$$
min_{x''} c''^Tx''
$$

where

$$
x'' = \begin{bmatrix} x_+ \\ x_- \\ s \end{bmatrix}
$$

and

$$
c'' = \begin{bmatrix} c \\ -c \\ \mathbf{0} \end{bmatrix}
$$


Subject to

$$
\begin{align}
A''x'' = b \\
x'' \geq 0
\end{align}
$$

Where $A'' = [A, -A, I]$

We are done. To summarize, we have proven the following statement:

Any optimization problem of the form:

$$
min_x c^Tx
$$

subject to:

$$
Ax \leq b
$$

has an equivalent optimization problem of the form:

$$
min_x c^Tx
$$

subject to:

$$
\begin{aligned}
Ax = b \\
x \geq 0
\end{aligned}
$$

# Farkas' Lemma as a Certificate of Infeasibility
Before we introduce the concept of duality, let's first notice something interesting about our second formulation. Our constraint is infeasible if b does not lie in the cone defined by the columns of A. This is trivial, as the set $\lbrace Ax \mid x \geq 0 \rbrace$ defines a cone, and our constraint explicitly says that $b$ must equal $Ax$ for some $x \geq 0$. From Farkas' lemma, we know that for any matrix A and point b, b lying in the cone of A's columns and there being a separating hyperplane between the cone and b which passes through the origin are strong alternatives. Thus, if we are able to find a vector $q$ such that $A^Tq \geq 0$ and $b^Tq < 0$, then the linear program must be infeasible, meaning no x satisfies the constraints. Once again, this is because q defines a hyperplane separating b from the cone defined by A's columns. We will refer to q as a *certificate of infeasibility*, meaning a proof that the optimization problem can't be solved.

### Certificate of a Lower Bound

Say that we have a LP in the first form we defined:

$$
min_x c^Tx
$$

subject to

$$
Ax \leq b
$$

If we showed that the optimization problem:

$$
min_x c^Tx
$$

subject to

$$
\begin{aligned}
Ax \leq b \\
c^Tx \leq t
\end{aligned}
$$

was infeasible, then we would know that t was a lower bound on how good our solution to the original optimization problem could be. This second formulation can trivially be formulated as a LP by making c a row of A and t an element of b. Once again, if this is infeasible then there would exist some vector $q'$ which would provide us a certificate of infeasibility. This would be a certificate of a lower bound on the original LP.

### Summary of LPs
At this point we have done the following: we have introduced the definition of a linear program, we've shown they can take two different forms, and we have shown that when a LP is infeasible Farkas' lemma tells us that there exists some certificate of infeasibility. This is all you must know to understand the next section.

## Duality
Duality confused me for a long time. We were all first exposed to duality long before we knew what it meant. If you've ever solved a constrained optimization problem using lagrange multipliers, you've used duality. But if you're like me, when you first used this method you had no idea why it worked. It wasn't until much later that I was actually taught why this method worked. It's because of duality. Even then, though, the intuitions behind duality did not stick. Duality lived in my brain as this nebulous idea; it was some sort of shadow clone of the original optimization problem, called the primal, and solving this shadow clone problem gave us information about the solution to the original problem. I stand by this definition. It is a correct one. But let's try to be more precise, at least for the case of a linear program.

### Dual of a linear program

Let's imagine we have a linear program in the second form we've defined. That is:

$$
min_x c^Tx
$$

subject to

$$
\begin{aligned}
Ax = b \\
x \geq 0
\end{aligned}
$$


Defining a LP with a lower bound can be written in this form as:

$$
min_{\{x, s\}} c^Tx
$$

Subject to:

$$
\begin{aligned}
\begin{bmatrix} A & 0 \\ c^T & 1 \end{bmatrix}
\begin{bmatrix} x \\ s \end{bmatrix} = \begin{bmatrix} b \\ t \end{bmatrix} \\
x \geq 0 \\
s \geq 0
\end{aligned}
$$

Note we are back in the form that Farkas' lemma talks about. Either there exists an $(x, s)$ which solves this equation or there is a separating hyperplane separating the cone defined by the columns of:

$$
\begin{aligned}
\begin{bmatrix} A & 0 \\ c^T & 1 \end{bmatrix}
\end{aligned}
$$

and the vector

$$
\begin{bmatrix} b \\ t \end{bmatrix} 
$$

If there is no (x, s) which solves this, then we can formulate the existence of the separating hyperplane as the existence of a vector

$$
\begin{bmatrix} y \\ \lambda \end{bmatrix}
$$

Such that

$$
\begin{aligned}
\begin{bmatrix} A & 0 \\ c^T & 1 \end{bmatrix}^T\begin{bmatrix} y \\ \lambda \end{bmatrix} \geq 0
\end{aligned}
$$

and

$$
\begin{bmatrix} b \\ t \end{bmatrix} 
^T\begin{bmatrix} y \\ \lambda \end{bmatrix} < 0
$$


When expanded out, these make the following set of conditions:


$$
\begin{align}
A^Ty + c\lambda \geq 0 \\
\lambda \geq 0 \\
b^Ty + t \lambda < 0 \\
\end{align}
$$

By dividing everything by $\lambda$, and letting $u = \frac{-y}{\lambda}$, we can rewrite this as:

$$
\begin{align}
A^Tu \leq c \\
b^Tu > t \\
\end{align}
$$

This tells us the following: if there is no feasible x such that $c^Tx < t$, then there exists some vector u such that:

$$
\begin{align}
A^Tu \leq c \\
b^Tu > t \\
\end{align}
$$

Finally, notice that for any feasible x, we have that $Ax = b$ and $x \geq 0$. Thus, 

$b^Tu = (Ax)^Tu = x^TA^Tu \leq x^Tc$ 

where the last ineqlaity is true since we know $A^Tu \leq c$ and the values of $x$ are all positive. This means that not only does such a u exist when there does not exist an x capable of making the objective drop below t, but it also means that any u satisfying these equations proves that no such x exists which could be feasible and have $c^Tx < t$. So, $b^Tu$ is a lower bound on how good the solution to the primal can be. It follows that the largest lower bound is the solution to the primal. Formulating the search for this largest lower bound we get:

$$
max_ub^Tu
$$

Subject to

$$
A^Tu \leq c
$$

which is simply a linear program written in the first form. This uses max instead of min, but we could simply negate b and say $min_u-b^Tu$. I leave as max since it's more informative of what the dual is accomplishing.

This is the dual of the LP. This is another linear program, where solving this LP will tell us the optimal objective to the primal. In this case the vector we are optimizing over is of size $m$, the number of rows in A. In our primal the LP is optimizing over a vector equal to the number of columns in A. So, in cases where there are a few constraints and a high dimensional vector space (meaning A has few rows but many columns), the space we are searching for an optimum shrinks drastically.

## Concluding Thoughts

There is a ton more to say about duality. This essay merely derives the dual problem for a linear program. I will likely write another piece on duality which aims to understand it more philosophically. Every optimization problem has a corresponding dual problem. Some are useful and some are not. The exact conditions which make the dual useful, the interpretation of the solution to the dual problem, and the applications of duality in statistics, optimization, and linear algebra which follow from solving the dual go very deep and are incredibly fascinating.

Farkas' lemma is also a powerful result in it of itself. We showed how it can provide certificates of infeasability in linear programs. But it can also be used to prove the existence of risk neutral pricing under no arbitrage assumptions. I wrote another piece [here](/writing/stiemke-theorem/) which proves this in column space using a closely related result called Stiemke's theorem, however Farkas' lemma can show the exact same result in row space.

#### Thanks
I want to give thanks to Stephen Boyd for teaching me everything I know about duality and linear programming. I also want to give thanks to my brother, Chris Healy, for providing insights into how Farkas' lemma can be used to prove risk neutral pricing in row space. Previously, I had only thought about this result geometrically from the perspective of the column space.