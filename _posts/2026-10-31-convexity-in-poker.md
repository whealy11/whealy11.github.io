---
layout: default
title: "Optimal Betting Frequencies Through the Lense of Convexity"
description: "This aims to connect concepts from poker theory to convexity, explicitly solving certain toy examples using simple linear programs. P.S. I host a private $5/$10/(strongly encouraged $20 straddle) game in Manhattan. If you're intrested in playing please email me. Pros are welcome as long as they are good vibes :)"
category: technical
image: /assets/images/convexity-in-poker/thumbnail.jpg
permalink: /writing/convexity-in-poker/
---

# Optimal Betting Frequencies Through the Lense of Convexity

**Will Healy**

## Introduction
The landscape of poker has changed drastically in the last decade. The game is much more quantitative now, especially online. Most live pros and even semi-pros would be able to give coherent explanations to once-niche concepts like minimum defense frequency. However, to be one of the best players it generally means spending your time studying specific spots, and using heuristics to reason about why a solver recommends a specific distribution over actions. This piece is meant to provide mathematically sound explanations for the relationships between specific spots. It does this through toy examples, and thus applying these concepts to make an actual decision at the poker table will take some work. However, even if they don't let you find some heroic bluff I think the concepts are quite interesting.

