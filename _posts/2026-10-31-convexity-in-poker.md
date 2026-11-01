---
layout: default
title: "Optimal Betting Frequencies Through the Lens of Convexity"
description: "This aims to connect concepts from poker theory to convexity, explicitly solving certain toy examples using simple linear programs. P.S. I host a private $5/$10/(strongly encouraged $20 straddle) game in Manhattan. If you're intrested in playing please email me. Pros are welcome as long as they are good vibes :)"
category: technical
image: /assets/images/convexity-in-poker/thumbnail.jpg
permalink: /writing/convexity-in-poker/
---

# Optimal Betting Frequencies Through the Lens of Convexity

**Will Healy**

## Introduction
The landscape of poker has changed drastically in the last decade. The game is much more quantitative now, especially online. But even live pros and semi-pros would now be able to give coherent explanations to once-niche concepts like minimum defense frequency. To be one of the best players it generally means spending your time studying specific spots, and building intuition on why a solver recommends a specific distribution over actions, with the goal of being able to later reason about similar spots. The goal of this piece is to provide mathematically sound explanations for why solvers behave the way they do, and describe relationships between specific spots. It does this through toy examples, and thus applying these concepts to make an actual decision at the poker table will take some work. However, even if they don't let you find some heroic river bluff next time you're playing, I think the concepts are quite interesting.

Let's start with some framing. Poker is a zero sum game of imperfect information. I remember hearing that exact sentence from this poker coach I studied from when I was a teenager. This means that in a no-rake environment the sum of money that the table starts with minus the sum of money that the table ends with is zero. The imperfect information part refers to the fact that there are things which you don't know, for example other players' hole cards and the cards which will come on future streets.

A poker strategy can be modeled as a deterministic function $f$ whose domain is the information a player could have accumulated in any given scenario and the output is a probability distribution over the legal actions available to them. To execute this strategy in a game you would simply sample from this multinomial of legal actions to choose the discrete action you play in the game. Thus, developing a good poker strategy essentially means learning a good function $f$.

Here a *good* function $f$ should be considered the one with a high expected return. Thus the *best* function $f^\*$ should be considered the one with the highest expected return. I think it's useful to divide the information that a player has at any given point in a poker hand into two categories: the concrete information $i_c$ and non-concrete information $i_n$. Concrete information refers to the information that would be written to a hand history database: the actions chosen by each player on each street, your hole cards, and the community cards. Non-concrete information refers to things which would not be written to a hand history database. These are things like your priors on certain players' behavior, how long someone took to make a decision, a slightly noticeable exhale of discontent from someone who folded pre-flop as as the flop comes 7h-4s-3c, etc. Non-concrete information is difficult to record and thus reason over mathematically. Concrete information is not. This piece will reason about the behavior of $f^*$ defined as:

$$
f^*(i_c) = max_f E[R_{i_c}(f(i_c))]
$$

where $R_{i_c}(q)$ is the reward from executing a strategy $q$. This means that $f^*$ is optimal over every single value piece of concrete information. I want to point two things out. 

The first is that this definition only really makes sense for river play, and technically only if you know you know it will be your last decision. This is the case if it was checked to you and you must go all in or check back. For a multi-street setting the reward of a policy at a given decision depends on what they would do in a future position. For example floating the turn may be a good idea in some scenarios, but not if your policy never bluffs rivers. To write out a definition of an optimal multi-street policy, you would copy notation from reinforcement learning and say that you are trying to maximize your expected reward over a randomly sampled rollout. I don't want to go this route, though, since this formulation implicitly fixes the transition probabilities of states. We don't want to fix these, but instead want to consider them dynamic adversarial, since they are influenced by our opponents tendencies.

The second is that calling $f$ deterministic and stating it's optimal for every single situation does not mean that we will not be *balanced*, a term thrown around a lot in poker. It simply means that $f$ itself encodes the balanced distribution, and that the sampling which is done to actually choose an action is where the balance comes from.

## Toy  Example 1

Let's consider a simplified poker scenario. Imagine we are heads up on the river, there are 100 dollars in the pot, and our opponent checks to us. Let's also imagine that we only have 100 more dollars in our stack and that we are committed to either going all in or checking. So if we bet our opponent will only have two options: call or fold. Further, let's assume that before looking at our cards there is a 50 % chance that we have a strong hand, a 50 % chance that we have a weak hand, and we know that our opponent has a middling hand: one that beats our weak hand but loses to our strong hand. Our strategy will then be defined as a vector:

$$
x \in [0, 1]^2
$$

where the first element is the frequency with which we are going to bet when we have a strong hand and the second element is the frequency which we are going to bet when we have a weak hand. Let's denote this strategy as the probability we value bet and the probability we bluff $x = [p_v, p_b]$.

Now let's consider our opponent's optimal strategy. Their strategy should maximize their expected profit. When we check, they don't do anything. When we bet, they have two options: call or fold. Thus their strategy's domain is a single element. They simply need to decide whether to call or fold when we bet. We can also assume that they know our strategy. If we consider money in the pot to be no-one's money at this point, their expected value of always folding when we bet is:

$$
R_f = (1 - p_b) * (0.5) * 100
$$

The $(1 - p_b) * (0.5)$ term is the probability that we have a weak hand and decided to check. The 100 refers to the size of the pot that they would win. This is the only scenario which they would win the pot, since when we bet with either a strong hand or weak hand they will fold and thus win nothing, and when we check with a strong hand they will also lose and win nothing. If they choose to call, their expected value is:

$$
R_c = (p_b) * (0.5) * 200 - (p_v)(0.5) * 100
$$

Where the first term comes from winning the 100 already in the pot plus the additional 100 that we used to bluff and the second term comes from calling their 100 dollar value bet with a losing hand.

Since we can assume that this opponent knows $p_v$ and $p_b$, then they will simply choose the maximum of these two vlaues. That is, their strategy would be to call if $R_c > R_f$ and fold if $R_c < R_f$. So, their expected value would be $max(R_c, R_f)$. Since poker is a zero sum game and thus every dollar that they are expected to lose becomes a dollar we are expected to gain, we want to minimize their expected return. Thus, our optimization problem when choosing $p_b$ and $p_f$ is:

$$
min_{p_v, p_b}max((1 - p_b) * (0.5) * 100, (p_b) * (0.5) * 200 - (p_v)(0.5) * 100)
$$

There's a common trick in optimization which is that if we have an optimization problem of the form:

$$
max_x(min(f_1(x), f_2(x)))
$$

is equivalent to:

$$
\begin{aligned}
max_{x, v} v\\
\end{aligned}
$$

Subject to

$$
\begin{aligned}
v \leq f_1(x) \\
v \leq f_2(x)
\end{aligned}
$$

So we can rewrite this as:

$$
\begin{aligned}
max_{p_v, p_b, v} v\\
\end{aligned}
$$

Subject to

$$
\begin{aligned}
v &\leq 50(1 - p_b) \\
v &\leq 100 p_b - 50p_v
\end{aligned}
$$





