---
layout: default
title: Separability
permalink: /notes/separability/
kind: prerequisite
---

# Separability

Separability refers to theorems and properties which make sets, points, subspaces, and objects separable, with an emphasis on linear separability. A linear separator is called a hyerplane, and is an n - 1 dimensional surface which divides an n-dimensional vector space into two halves. In $R^2$, a hyperplane is a line. In $R^3$, a hyperplane is a plane. For higher dimensions, these surfaces are hard to visualize but share the same properties. A halfspace refers to a portion of a vector space which lies on one side of a hyperplane. A hyperplane is typically defined by two values $c$ and $k$, where $c \in R^n$ and $k \in R$. It is defined as:

$$
\{x | c^Tx = k\}
$$

The set of points whose dot product with a certain vector equals a constant draws a surface perpindicular to the vector. The value k translates that surface along the vector, with $k = 0$ having the surface pass through the origin.

A halfspace is often defined as:

$$
\{x | c^Tx \leq k\}
$$

Replacing $\leq$ with $<$, $>$, or $\geq$ also yield valid halfspaces.

## Separability of Disjoint Sets
Not all pairs of disjoint sets are strictly separable. This is not surprising. The set of all pairs of disjoint convex sets comes closer to being strictly separable, but still is not a sufficient condition. Take for instance the two disjoint and convex sets:

$$
C = \{(x, y) | y \geq e^x\}
$$

$$
D = \{(x, 0) | x \in R\}
$$

As $x \to -\infty$, $e^x \to 0$. Thus, these two sets become infinitely close and any hyperplane aiming to separate them would fail, since if the hyperplane sat any distance above the x-axis (which it needs to) it will eventually cross the line $e^x$.

### Sufficient Condition for Separability
In order for two convex sets to be separable, at least one set must be compact, and both sets must be closed. A compact set is one where the set has finite bounds and contains its bounds. A closed set is one which contains its bounds. We will lean on this condition later on.