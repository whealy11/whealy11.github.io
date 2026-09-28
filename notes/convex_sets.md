---
layout: default
title: Convex Sets and Cones
permalink: /notes/convex-sets/
kind: prerequisite
---

# Convex Sets and Cones
A set is convex if it does not contain any two points such that the line segment connecting them contains a point not in the set. For example a circle is a convex set. A star is not a convex set, since the points between two tips of the star lie outside of the star. Convex sets can be bounded or unbounded. A circle is a bounded convex set. $R^n$ is an unbounded convex set. 

A cone is type of convex set defined as 

$$
\{Ax|x \geq 0\}
$$

It is literally shaped like a cone. Take the cone created by vectors $\begin{bmatrix} 1 \\\\ 1 \end{bmatrix}$ and $\begin{bmatrix} 1 \\\\ 0 \end{bmatrix}$. The cone spanned by these two vectors looks like this:

<img src="/assets/images/cone-example-11.png" alt="Cone spanned by (1,0) and (1,1)" class="figure-sm">

### Non-negative Orthant
The non-negative orthant is the simplest cone. It is defined as

$$
\{x | x \geq 0\}
$$

In $R^2$ this is simply the first quadrant.

