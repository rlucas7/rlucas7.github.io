---
title: 'Notes on papers regarding block Kaczmarz algorithms'
date: 2025-07-17
permalink: /posts/2025/07/blog-post-3
tags:
  - algorithms
  - probability
  - linear algebra
  - paper-notes
---

# Summary

These few papers are the existing literature of work on block Kaczmarz. There's a lot of opportunity in the space of applying this to AI & ML problems.

# block Kaczmarz,a central premise

The primary difference between block and non-block Kaczmarz algorithms, the number of rows used on each step is either >1 (block) or equal to 1-that's it.
However, rows greater than 1 brings a subtle amount of richness and complexity that is non-obvious to those not studying linear algebra in detail.
In this post we'll look at a few papers that study block Kaczmarz in detail.

# Some subtler points


# The Bucharest Connection

Basically if you can only read one paper on block Kaczmarz read Ion Necoara's paper.
In [Faster Randomized Block Kaczmarz algorithms](https://arxiv.org/abs/1902.09946) Necoara does a couple of things:

1. Relates convergence to the size of the blocks (blocks are the number of rows selected)
2. Relates convergence to the geometric properties of the blocks
3. Analyzes extrapolation methods for block Kaczmarz
4. Provides an analysis of a block method that exploits current parallel architectures on computers (gpu and cpu).
5. Analyses importance measures across blocks (using ideas from probability)

There are a fair number of details in the paper and it's well worth the time investment, I'll only mention the key points in the proofs
that are interesting.

One interesting item is the proofs enable unified analysis of static and adaptive step sizes.
Here step sizes are \( \alpha_k \) from the non-block case and if you set these equal to the same value for all \( k \) then it's not adaptive.
The python code from the previous blog post on Kaczmarz row action methods for a single row at a time all used non-adaptive step sizes.

The starting idea, use Courant-Fischer to bound the operator matrix norm above the smallest eigenvalue, this part is similar to the proof in the non-block case.
For some probability measure, select a subset of the rows of the matrix. There are multiple subset measures possible, the paper explicitly mentions two; uniform across rows, the second is uniform across the sets that constitute the partition. The paper then shows a case where these two probability measures are equivalent.
The proof clearly explains what seems intuitive, the largest eigenvalue of the sampled blocks is largely governing the dynamics of whether a block approach is faster or not.
In particular the author's choice of uniform probabilities makes it easier to have a single value to compare the largest eigenvalue of the sampled blocks against.
If the probabilities were not uniform then I suspect you'd need a measure that summarizes the divergence or the distance between the uniform and the non-uniform as then you're comparing two moving quantities, the probabilities and the eigenvalues of the blocks, making things more complicated. This is why Necoara does the argument this way, to keep things simple and therefore make progress.

That being said, one weird thing that seems to continue is the use of uniform partitions of the rows in the partitioning simulations.
For example each subset of rows is \(\{J_i : \kappa(l), l = \lfloor (i-1)\frac{m}{l}\rfloor + 1, \dots, \lfloor\frac{m}{l} \rfloor \} \)  where \( i=1,\dots, l \).
There are of course other ways to do this but why do I think this way is a bit weird? The reason is that the blocks get selected in situ, e.g. they're selected as they are structured already and if you had some structure it could lead to artifacts. However, these seem like unlikely events and can be easily mitigated by first independently permutating the rows before any partition sampling. Now the reason why this approach to partition sampling is chosen seems to be because of the literative on row pavings.

Row pavings that are called 'good' have the property that all partitions subsets have approximately equal size and this allows you to bound the largest eigenvalue.
I'm not super familiar on the subject of row pavings but this result is derived in [this paper](https://tropp.caltech.edu/conf/Tro09-Column-Subset.pdf) by Tropp which is reference 26 in the Necoara paper.

The other very interesting item in this paper is the proof of Theorem 5.1, the Chebyshev step sizes.
What is interesting here is that the Chebyshev step sizes polarize the largest and smallest eigenvalues.
Also, the minimax property of the Chebyshev polynomials is what driver the proof.
In essence the norms of the matrices convert the orthogonal projections into a polynomial in the matrices.
This polynomial then has the maximum value minimized as a uniquely defining characteristic of Chebyshev polynomials.
Of course this is fairly standard for scalar numerical approaches but I've not seen it used to such a striking effect with Matrices before.
Basically the proof takes a one setp recursion backwards on the projects and then iterates and unwinds the recursion to get a polynomial with matrix argument.
Why do we care? In theory, and often in practice, the Chebyshev based stepsize will give the fastest algorithms.
The extrapolating step sizes chosen by the Chebyshev polynomails are a form of momentum method where there is a principled mathematical argument for the choice of size.
All other proofs I've seen for momentum methods before usually rely on physics intuitions, and while useful, are also somewhat limiting in their applicability.
The momentum method is used broadly and it's often difficult to justify the physics intuition in a particular application, the Chebyshev argument liberates
us from this requirement.

This also seems to be related to a matrix inequality known as Meany's inequality.

The importance of finding well conditioned blocks is also illustrated in the proof. Well conditioned blocks correspond to the 'good' pavings, e.g. partitions of the rows
into subsets where each submatrix has roughly equal largest eigenvalue.

## Exploiting parallel architectures

this explains all the 5 points above except the fourth one, how does this method allow exploiting of parallel architectures?
For the explanation, we need to only look at the randomized block Kaczmarz algorithm (RBK) in section 2.2 of the paper, the iterates are defined as

$$
x_{k+1} = x_k - \alpha_k \left(\sum_{i \in J_k}\omega_{i,k} \frac{a_i^Tx_k - b_i}{\|a_i\|^2}a_i\right),
$$
where \( \omega_{i,k} \in [0, 1] \) and \( \sum_{i \in J_k} \omega_{i,k}=1 \). So to interpret, this means we calculate the row actions each individually, similar to how
we would do with a non-block approach and then take the weighted average of these row actions. If each block size does not exceed the number of processors or threads then we can effectively leverage parallel architecture because each row action can be computed in paralle. The beautiful part here is how the theory of row pavings interlocks with modern parallel processing architures to enable faster algorithms.

## When to use the Block methods studied here?

If you can construct an efficient paving that has good conditioning properties then it is worthwhile to look into block methods.
This is especially true if you have a compute architecture which can leverage parallelism to a large degree, then the averaging
approach described by Necoara is worth looking at in detail.

Ok so if you read this you might think, ok problem solved what else is there? Well in higher dimensions there are many ways to define probability measures.

# Another probability measure, volumes

This second paper [Randomized Block Kaczmard with Volume Sampling: Momentum Acceleration and efficient implementation](https://arxiv.org/pdf/2503.13941)
looks at a different approach to the multi-dimensional probability measure over subsets of rows of the matrix A.

The Authors use subsets of A and form A_SA_S^T and sample proportional to the volume of these submatrices relative to the full matrix.
This is a valid probability measure and they show an interesting algorithm to perform this computation efficiently. They are essentially doing
a prefix sum over a clever structuring of the sub-matrices and using determinant identities to facilitate the computation.

The other interesting thing they mention here that wasn't mentioned in Necoara's paper is relaxing the notion of row paving (partitions) to row coverings.
Row coverings are no longer partitions of rows because they allow subsets of the rows to have non-empty intersections. The authors relate the volume sampling to
determinental point processes, a technique popular in the machine learning literature.
The authors of this paper use a slightly different mathematical argument to Necoara.
Instead of using Chebyshev polynomials, they use Symmetric polynomials, effectively a different basis.

Two things that are prevalent in their proofs and not seen in the other papers I've read:

1. They get a quantity that allows them to quantify the tradeoff of different block sizes,

$$
\frac{K_{s_1}}{K_{s_2}} = \frac{\sum_{i=s_1}^{rank(A) \sigma_i^2(A) }{ \sum_{i=2_2}^{rank(A) \sigma_i^2(A)},
$$
where \( 1 \leq s_1 < s2 \leq rank(A) \) and \( \sigma_i^2(A) \) are the singular values of the matrix \( A \).
In certain cases of random variables the distribution of this fraction will be a beta distribution which is interesting to note because the closed form
distribution is readily available and the two parameters control the shape of the density and therefore allows us to quantitatively validate the speedup claims.
I didn't see the authors exploiting this relationship in their simulation studies though.

As an aside here's the details, if you'd like an indication of how this would work as a proof.
Take the QR decomposition of a matrix of standard Gaussians. The R matrix will then have Chi-squared distribution for the diagonal elements, so \( Q^Tb \) is then
the right hand side, of the system of linear equations. The eigenvalues here are independent random variables and the distribution of the ratio of the two will have a beta distribution. What does this get you?
It gives you a somewhat structural understanding of how the choice of \(s_1 < s2 \) will impact your simulations if you sample \( A \) according to a Gaussian distribution.
Incrementing/decrementing \( s_1 \) or \( s_2 \) can have a large impact on the shape of the distribution in certain regimes.
Also for large rank matrices \( A \) then the continuous values of the beta random variable can be a helpful metaphor as  a cognitive tool.

## And now some more notes ...

2. The proofs for the determinantal point process, or volume sampling, have ratios of sums of determinants of submatrices so formulae and proofs appear which
leverage the [Cauchy-Binet formula for determinants](https://en.wikipedia.org/wiki/Cauchy%E2%80%93Binet_formula) appear prominently in the derivations.

The authors also have a momentum variant and provide proofs of convergence using homogeneous linear recurrence relations (section 4 of the paper).

Section 6 of this paper has lots of numerical results and simulation studies that are worth reading if you plan to use a block Kaczmarz method.
There is also a connection to gossip protocol/algorithms mentioned in this section as well. A graph consensus application is studied.

## When to use

The exact circumstances will vary but if the A matrix is static or fixed and the b vector is varying across problem instances then it
can be the case that the preprocessing required for the Cauchy-Binet style pre-computation is not too onerous over all b vector instances.

# Gower and Richtarik

The other work here that is a must read is [Gower and Richtarik](https://arxiv.org/abs/1506.03296) which looks at Kaczmarz from 6 different perspectives.
All useful perspectives and all providing new insight into the problem.

The 6 viewpoints:

1. sketching
2. optimization
3. geometric
4. Algebraic-deterministic
5. Algebraic-random
6. Analytic-random fixed point

The paper elucidates on each one of these viewpoints, I've included the equation that corresponds to each viewpoint here in the post.
If you've worked on problems in this space before, or you read my previous post of row action Kaczmarz, then these perspectives are hopefully clear from the equations.

I will note that the paper is not strictly a block Kaczmarz paper but a perspectives paper, elucidating how the different perspectives suggest and lead to different algorithms and different complexity results.

This paper is worth reading in addition to the paper of Necoara mentioned above if you haven't already read this paper and are interested in these methods.
