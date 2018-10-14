---
layout:     post
title:      Solving the canonical consumption-saving model with Haskell
date:       2014-06-09 12:32:18
summary:    A short demo to show that functional languages can be a powerful tool to numerically solve economic problems
categories: Haskell macroeconomics VFI
---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

## Why Haskell

Some text about why Haskell is awesome...
Some text about why Haskell is awesome...
Some text about why Haskell is awesome...

## The model

We will solve the planner's problem. The planner wants to maximize.

$$
\begin{align*}
\max& \sum_{t=0}^\infty \beta^t u(c_t) \\
s.t. \ & k_{t+1} = f_{k_t} + (1-\delta)k_t - c_t \\
       & k_{t+1} \geq 0 \\
       & k_0 > 0 \text{ given}
\end{align*}
$$

We have the usual assumptions on $$f$$ and $$u$$, so this problem is well defined and has a unique solution.

Now let us rewrite the problem in dynamic programming terms. Let

$$
V(k) = \max_{k'} \left\{ u[f(k) + (1-\delta)k - k'] + \beta V(k') \right\}
$$

It is not difficult to see that the two problems are equivalent. It can also be shown that the right hand side is a contraction mapping, therefore it has a unique fixed point, the value function we are looking for. Also, starting from any $$V_0$$, this fixed point can be found by iteration.

## Implementation

### Types

Haskell has a powerful type system, so we will utilize it to make our code easier to understand. As we will see, with properly choosen types, the function signatures will say a lot about what a function does. So let us define our types

{% highlight haskell %}
type Consumption = Double
type Utils       = Double
type Capital     = Double
type Parameter   = Double
{% endhighlight %}

We will use a grid of `(Capital, Value)` pairs to represent our discretized value function

{% highlight haskell %}
type ValueGrid   = [Utils]
type CapitalGrid = [Capital]
{% endhighlight %}

### Building blocks

Now with the types done, let us write the basic building blocks of our model: the grids and the utility and production functions.

The grids will be represented as simple lists.

{% highlight haskell %}
kGrid  = [0.1, 0.2 .. 10] :: CapitalGrid
vGrid0 = map log kGrid    :: ValueGrid

{% endhighlight %}


And we will use CRRA utility and Cobb-Douglas production functions.

{% highlight haskell %}
-- CRRA utility function
crra :: Parameter -> Consumption -> Utils
crra 1 c = log c
crra g c = c ** (1-g) / (1-g)

u :: Consumption -> Utils
u = crra 1.5

-- Cobb-Douglas production function
fCD :: Parameter -> Capital -> Capital
fCD a k = k ** a

f :: Capital -> Utils
f = fCD 0.8
{% endhighlight %}

### Interpolation

As $$V$$ is represented by a grid, we will need to find $$V(k)$$ using interpolation if $$k$$ does not happen to be a gridpoint. For this we will use the `numeric-tools` package. If you do not have it, open a command prompt or terminal and type:

{% highlight terminal %}
cabal update
cabal install numeric-tools
{% endhighlight %}

