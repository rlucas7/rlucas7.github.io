---
title: 'Gearhart-Koshy-Acceleration of Kaczmarz'
date: 2025-07-23
permalink: /posts/2025/07/gearhart-koshy-acceleration
tags:
  - paper-notes
  - gearhart-koshy
  - kaczmarz
  - line search
  - optimization
---

# Kaczmarz Algorithm

This is the third post in a series of posts on Kaczmarz algorithm.
Previous blog posts are

<a href="https://rlucas7.github.io/posts/2025/07/Kaczmarz-row-actions" aria-label="Click here to read the first post, focusing on kaczmarz">Intro row-action methods, Kaczmarz (post #1)</a>
and
<a href="https://rlucas7.github.io/posts/2025/07/block-kaczmarz" aria-label="Click here to read the second post, focusing on block kaczmarz">Block Kaczmarz (post #2)</a>,
and these describe single row and block Kaczmarz algorithm papers respectively.
If you're not already familiar with Kaczmarc and the block variants, feel free to check those posts out as well.

# Gearhart Koshy Acceleration

Let's start with line search, what is a line search problem?

While there are some exceptions, most optimization routines rely on having gradients readily available.
Gradients are handy for optimization routines because, for convex functions, steps in the negative direction of the gradient
will yield a decrease in the objection function.

$$
f(x_k + \alpha_k g_k).
$$
Here notice that \\( g_k \\) is the gradient at the point \\( x_k \\) and \\( f(\cdot) \\) is the objective function we're trying
to optimize. For this problem if \\( g_k \\) and \\( x_k \\) are known then \\( \alpha_k \\) is the only unknown and this function
is a scalar function in \\( \alpha_k \\). Also we said \\( f( \cdot ) \\) is convex so this means that there is at least 1 minimal
value inside a range \\( [a,b] \\) for \\( f( \cdot ) \\). One of the oldest and most robust methods for solving this is binary search.
Geometrically though, the collection of points for \\( \alpha_k \in [a, b] \\)  of \\( x_k + \alpha_k g_k \\) form a line, or a line segment
if you are a pedant. There are many ways to solve this problem beyond binary search, there is a golden section search method and
other methods too, even if \\( f( \cdot ) \\) is non-differentiable in the interval you can use the
<a href="https://en.wikipedia.org/wiki/Nelder%E2%80%93Mead_method" aria-label="Click through to read the wikipedia page for the Nelder-Mead method">Nelder-Mead method</a>.

So line searches are fundamental subproblems in many optimization routines.

Now that we understand the problem let's look at a basic dichotomy in line search problems, exact vs inexact methods.

# Exact vs Inexact

For a convex function, if we suppose the function is strictly convex then there is a unique minimal point along the interval for the line search problem.
This because we are taking the minimal point on each step the algorithm is strictly decreasing and therefore must converge to the minimal point.
A subtle point is that we need not take a full step to the minimal point on each line search sub-problem.
You might be asking yourself:
"Why would we want to do such a thing?", and I'll tell you that beside coming up with algorithms to find optimal points, applied researchers want to solve actual problems too.
In cases where doing the line search to the minimal point is too computationally expensive, say in high dimenional \\( x_k \\) spaces, it's often easier computationally to
take a step in the right direction rather than the step to the smallest spot. In these scenarios you're using an *inexact* line search method.
These can also be proven to converge and I'll skip the details of that convergence proof here.

# Basic Idea

Now that we've gotten line search problems out of the way and we've got an intuitive understanding of line search methods the next question is what does this
have to do with Kaczmarz? Let's delve deeper to find out the details.

We learned back in the first blog post of the series how the Kaczmarz algorithm works for single rows.
We also saw that graphically what is happening for each row is that the Kaczmarz iteration does an orthogonal projection onto the other hyperplane(s).
Each of these steps can be formulated as project operators \\( P_j \\).
Now as long as there is a non-empty intersection of the projection spaces then the Kaczmarz algorithm will converge to a point in the space of intersections.
In our simple examples in 2-dimensions the point where the two lines intersected was that space.

Now let's look at the series of projections, denoted \\( Q_j = \prod_{i=1}^j P_i \\), these will be the cumulative action of the Kaczmarz projections.
The line search problem then becomes

$$
t_k = \textnormal{min}_{t\in\mathbb{R}} \|x_k + t(Q_{n-1}(x_k) - x_k) - P_n\|^2,
$$a repea
where here we minimize in \\( t \\). This is a quadratic function in \\( t \\) and has a minimal point.

The contribution of the three papers here look at:

1. Scalar methods extended from linear spaces to affine spaces.
2. Generalized methods that work on block Kaczmarz and how these relate to volume sampling.
3. The Krylov subspace perspective on the Generalized method.

# Notes On Scalar method

Let's first discuss the difference between *linear* spaces and *affine* spaces in Linear algebra.
The difference is subtle but significant. In affine spaces you allow constants on your functions and in linear spaces you force the constants to be 0 always.
One reason why this matters has to do with vector spaces and subspaces, they need to include the origin to satisfy an axiom known as closure.
If you allow any vector to be offset by a non-zero value then you need to remove that offset to ensure that all linear spaces include the origin.
While you might say this is me being a pedant I assure you I'm not, this goes back a few hundred years in the math world at least.

As an aside if you're familiar with
<a href="https://en.wikipedia.org/wiki/Legendre_transformation#Geometric_interpretation" aria-label="Click here to read the geometric interpretation of Legendre-Fenchel transform on wikipedia">Legendre-Fenchel transforms</a>
then you'll notice how the derivative of the function encodes the dual and ignores any constant offset terms. Here the linear spaces idea is similar.

Now some things that apply in linear spaces also apply-suitably modified-in affine spaces as well. The Gearhart-Koshy acceleration is one of those things.
The mathematical content of the first paper <a href="" aria-label=""></a> amounts to extending previous results from linear spaces to affine spaces.
This means that the techniques of the line search can be applied to affine spaces too. I'll refer you to the paper for the details of the proofs.

Let's look at the Linear
$$
t_k = \frac{\langle x_k - Q_j(x_k), x_k \rangle}{\|x_k - Q_j(x_k)\|^2},
$$
and compare this against Affine,
$$

t_k = \frac{1}{2} + \frac{\sum_{j=1}^n\| Q_j-1(x_k) - Q_j(x_k) \|^2}{ 2 \|x_k - Q_j(x_k)\|^2},
$$
and you can see that the inclusion of the one-half term and the summation in the numerator is the primary difference.
An interesting line of inquiry would be to determine a probabilistic proof of this equation.
My hunch is that the one half term can be explained by an expectation over a suitable probability measure for the space.

The paper also includes some computational examples which study varying the Friedrich's angle between the subspaces which acts as a bound on the convergence rate.
The smaller the Friedrich angle, the faster the convergence of the Gearhart-Koshy method when compared against a non-accelerated method.

# Notes on Block Gearhart-Koshy

In
<a href="https://rlucas7.github.io/posts/2025/07/block-kaczmarz" aria-label="Click here to read the second post, focusing on block kaczmarz">post 2</a>,
of the series we looked at a couple methods that partition the rows and then do Kaczmarz updates over these subsets.

One of the papers surveyed in that post looked at what is called volume sampling.
Volume sampling defines the probability as the relative size of the determinants of the sampled block submatrix to full matrix.

In that prior paper the focus was on was to analyze the block Kaczmarz and understand when the block method accelerates and when it is overly
complicated without improving runtime, or is potentially numerically unstable.

Here the idea is slightly different, Janosch Rieger in
<a href="https://arxiv.org/abs/2201.10118" aria-label="Click to read the abstract on the arxiv page for the preprint">Generalized Gearhat-Koshy acceleration for the Kaczmarz method</a>, shows how the Gearhart-Koshy acceleration method can be applied to block Kaczmarz and the minimization becomes over the distances between the subspaces of the blocks.

The formulae that come out of the analysis are volume sampling formula, sorry for the spoiler. It's a super interesting result though because
it says something about the volume sampling approach for blocks, it's also an acceleration method.

# Krylov Subspaces and Gearhart-Koshy

The final of the three papers to discuss on Gearhart-Koshy acceleration methods for Kaczmarz is the paper
<a href="https://arxiv.org/abs/2311.18305" aria-label="Click here to navigate to the arxiv landing page for the Gearhart-Koshy Krylov subspace method preprint">Generalized Gearhart-Koshy acceleration is a Krylov subspace method</a>.

This paper looks at the Kaczmarz method and looks at the Gearhart-Koshy acceleration. The authors show that for a square and linear system, the method works in exact arithmetic. Nothing numerical analysis style here. They show that there is also a Gram-Schmidt style orthogonalization approach and that both the Krylov perspective and the Gram-Schmidt perspective converge in typical problems.

Note that here the vector spaces here must be linear and not affine. Krylov subspaces are *linear* subspaces of the vector space.
