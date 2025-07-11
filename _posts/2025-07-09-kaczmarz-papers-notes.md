---
title: 'Notes from Several papers on Kaczmarz algorithm'
date: 2025-07-09
permalink: /posts/2025/07/blog-post-1
tags:
  - Linear Algebra
  - Randomized Algorithms
  - paper-notes
---

# Kaczmrz Algorithm and Related Algorithm Notes

# Paper 1
The paper [The Kaczmarz algorithm, row action methods, and statistical learning algorithms](https://faculty.sites.iastate.edu/esweber/files/inline-files/KaczmarzSGDrevised.pdf)
s a Contemporary Mathematics, 2018 paper that looks at both Kaczmarz and related problems.

## Key points of the paper

1. Relaxations of Kaczmarz can be solved via Hildreth, a variant of Kaczmarz.
2. Projection onto Convex Sets (POCS) where the projections are onto rows of the `A` matrix (affine hyperplanes).
3. Schwarz iterative method (aka subspace correction method) for solving symmetric positive semidefinite (SPSD) linear systems
over separable Hilbert spaces are a generalization of Kaczmarz
4. Randomized Kaczmarz is a special case of SGD w/minibatch size 1 and the objective function is the square of the affine hyperplane.

Note that for the LA approaches you can usually apply Kaczmarz to submatrices given a matrix partition.
Partitioned approaches are *not* discussed in this paper.


# Basic Idea of Kaczmarz

You've got a system of linear equations (SLE) and you want to solve them,

$$ Ax = b $$

here \(( A \)) is a \((rows \times cols \)) matrix and \(( x \)) is a column vector, as is \(( b \)).

Of course there are several ways in linear algebra (LA) to solve this problem.
\(( A=LU \)) and \(( A=QR \)) using the LU and the QR decompositions come to mind.
Both of those approaches are deterministic as typically presented to students.
We will follow a similar approach with Kaczmarz, starting from the deterministic
version and progress into randomized etc. versions.

$$
x_{k+1} = x_{k} + \alpha_k \frac{b_{i_k} - \langle a_{i_k}, x_{k} \rangle}{\| a_{i_k} \|^2} a_{i_k},
$$
where \(( \alpha_k \in (0, 2) \)) for convergence.


Now there is nothing about this that is random as currently written.
For an algorithmic procedure we start with an initial guess \(( x_0 \)) and then update until a convergence criteria
is met. This might be a change in \(( x_k \)) on successive iterates, or a maximum number of iterations.
These are two common stopping criteria.

Now we said that this is an SLE but if we instead have inequalities (abbr. SLI) then we handle the
solving routinte very similarly

$$
c_{k+1} = \textnormal{min}\left{0, \alpha_k \frac{b_{i_k} - \langle a_{i_k}, x_{k} \rangle}{\| a_{i_k} \|^2}\right},
x_{k+1} = x_{k} + c_{k+1} a_{i_k}
$$
where \(( \alpha_k \in (0, 2) \)) for convergence.
The difference here is that if we are on the boundary of the affine hyperplane then we need to stop moving which
forces the \(( c_k=0 \)) for the iteration. If non-zero, then there is some flexibility in movement and the
\(( x_k \)) vector will meander in a meaningful manner.

At this point it may seem like a bunch of formulae and calculations but what is the intuition of why this works?

## Why Kaczmarz works

Let's recall the cosine of the angle between two lines, in LA terms this is written

$$
\frac{\langle a, x \rangle}{\| a \|^2} = cosine(\theta),
$$
where here \(( \theta \)) is the angle between the two lines, \(( a \)) and \(( x \)).
That the lines are defined by vectors can be visualized in 2-dimensions where the point in 2 space represents a line emanating from the point
\(( 0, 0 \)) which is often ominously called `the origin'.

Let's have some visualization in the Figures 1 & 2.

<img src='{{site.baseurl}}/images/Kaczmarz_Simple_Sequential.png'/>
<img src='{{site.baseurl}}/images/Kaczmarz_Deterministic_Figure2.png.png'/>


To keep things simple I use square 2-dimensional matrices so there are only 2 lines to show.
The dashed red line indicates the direction of the movenment of each \(( x_k \)).
These solutions converge on the exact solution, though the progress of the deterministic
Kaczmarz decreases with each iteration.
You'll notice that each successive red dot (an \(( x_k \)) ) is a projection from the previous point
orthogonally onto the other line. This behavior holds in higher dimensions as well and if we have a tall
matrix instead of a square one we would cycle projections over the 3 or more lines of the matrix, each being an
orthogonal projection.

# Kaczmarz With Momentum

The paper [RANDOMIZED KACZMARZ WITH GEOMETRICALLY
SMOOTHED MOMENTUM](https://arxiv.org/pdf/2401.09415) is an obvious approach after having visualized the deterministic
teps in the plots.

[Momentum methods](https://distill.pub/2017/momentum/) have been around for awhile and the nice thing that they
do is take into account these geometries inherent in the space you're trying to manoever.

The momentum update the authors use is

$$
x_{k+1} = x_{k} + \frac{ b_{i_k} - \langle x_k, a_{i_k} \rangle}{\|\|a_{i_k} \|\|_2^2} a_{i_k} + M y_k,
$$
and
$$
y_{k+1} = \beta y_k + (1-\beta) (x_{k+1} -x_{k}),
$$
where here \(( M \in [0,1] \)) and \(( \beta in [0, 1) \)). If we set \(( \beta = 0 = M \)), then we get the
usual Kaczmarz algorithm and the \(( y_k \)) tracks the distance between successive points in the iterations.


<img src='{{site.baseurl}}/images/Momentum_Kaczmarz_Figure.png'/>

The authors of this work also setup a formulation of Kaczmarz that uses Nesterov Acceleration (NA) .
NA has 3 equations and I'll refer you to the paper to read those.
They also have a nice theorem that gives quantitive information on the iterations of a randomized NA approach.

In addition, they study the relationship between the eigenspace of the problem and the parameters of the
acceleration methods, both Momentum and NA.

The paper is very well written and contains many empirical examples to develop intuitions on the methods.

# Greedy Kaczmard

The paper [Convergence Rates for Greedy Kaczmarz Algorithms,
and Faster Randomized Kaczmarz Rules Using the Orthogonality Graph](https://www.auai.org/uai2016/proceedings/papers/77.pdf)

Looks at two row selection rules that are deterministic but greedy and update as we move through the space.
In the simple Kaczmarz algorithm we update (project) onto rows \{0, 1, ..., # rows - 1\} successively.
We then go back and do the full pass again and again until stopping criteria are met.

With the greedy approach they use the following rules

$$
 i_k = \textnormal{argmax}_i \| a_i^Tx_k - b_i \| \ \ \textnormal{MR},
$$
or
$$
 i_k = \textnormal{argmax}_i \| \frac{a_i^Tx_k - b_i}{\|\| a_i \|\|} \| \textnormal{MD},
$$
where the MR and MD are for maximal residual and maximal distance, the distance being between \(( x_k \)) and \(( x_{k+1} \)).
The thing I like about these row selection rules is that they *learn* as the algorithm progresses.
This learning comes from the inclusion of the \(( x_k \)) values in the calculation.
The tradeoff is that the learning comes at a cost, for each update when we learn new row entering ranks, we incur sortation cost of \(( \mathcal{O}(rlog(r)) \)) where \(( r \)) is the number of rows in the matrix \(( A \)).

This paper also mentions that SLI can be solved with the greedy approach too.
By now we know that this is pretty straightforward.
One interesting note they mention is that you can do any sort of norm optimization with a constraint with the SLI approach.
The paper mentions using the SLI with a binary search over \(( \tau \)) to solve

$$
\textnormal{min}_{x} f(x) = || Ax - b ||_1 + \lambda || x ||_1, \lambda \geq 0.
$$
And using \(( f(x) \leq \tau \)) where we vary \(( \tau \)) on each iteration, basically transforming the L_1 constrained L_1 optimization problem into a univariate binary search
and using the Kaczmarz as a subroutine.

# Application To GUI Layouts

The manuscript by []() gives several applications. One that I think is cool is to GUI layouts with hierarchies.
You can think of CSS flexbox as an example but the Auckland model is more layered IIUC.

<img src='{{site.baseurl}}/images/constraints_ui_layout_figure.jpg'/>

The figure gives you an idea of the mixture of hard/equality vs soft/inequality constraints that go into setting up
a visualization layout of widgets.

The hierarchy in the Auckland layout model is a list of floats that are application specific and determined outside of
the Kaczmarz algorithm.

As with any linear program style problem to have an initial starting point is vital.
If you don't have a feasible/valid starting point then phase I/II methods are the classical ways to find a starting point.
You can also use this Kaczmarc method too, if you know how these phase I methods work.

# Irreducible Infeasible Sets (IIS)

To solve an optimization problem with an iterative routine you typically need to have a valid starting point.
The identification of excessive constraints so that the problem has no valid starting point is called the IIS problem and
was introduced in [Gleeson and Ryan](https://www.math.ucdavis.edu/~deloera/MISC/LA-BIBLIO/trunk/RyanJennifer2.pdf) and also studied by [Chinneck and Dravnieks](https://www.sce.carleton.ca/faculty/chinneck/docs/ChinneckDravnieks.pdf) an ORSA Journal of Computing paper from 1991.

The solution in the Chinneck and Dravnieks paper is a deterministic procedure conceptually related to the [Big-M method](https://en.wikipedia.org/wiki/Big_M_method).
Using the ORM or Hildreth approach, [Jamil, Chen, and Cloninger](https://arxiv.org/abs/1409.2902) look at Graphical user interface (GUI) layouts with hard and soft constraints and compare the randomized approaches finding large speedups.

An alternative to the IIS is the MFS problem where you seek to find the largest subset of linear constraints such that the feasible set of points is non-empty.

The application not withstanding, the approach in the paper may be leveraged for identification of IIS constraint sets which is a useful tool for a user
who is running an optimization routine and encounters an empty (infeasible) constraint set. For large scale systems of affine hyperplace constraints, this approach
is like a flashlight in the dark, telling the user which constraints to inspect and possibly remove.

An open source benchmarking dataset for IIS is available from [netlib](https://www.netlib.org/lp/infeas/readme).

# Survey paper

The paper [Survey of a class of iterative row-action methods:
The Kaczmarz method](https://arxiv.org/pdf/2401.02842)
has even more variants of this algorithm. The manuscript includes applications and parallelization strategies.

# Stay Tuned For More

In the papers surveyed in this post all the Kaczmarz algorithms were single row action methods.
In the row action algorithm nomenclature, this means they operate on one and only one row of the A matrix at a time.
There are also variations that look at more than a single row. I'll put those up in a future post though to keep things
from getting too long here.


<script src="https://giscus.app/client.js"
        data-repo="rlucas7/rlucas7.github.io"
        data-repo-id="MDEwOlJlcG9zaXRvcnkzODMyNTM2MzA="
        data-category="General"
        data-category-id="DIC_kwDOFtf8fs4Cqeqt"
        data-mapping="url"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="en"
        crossorigin="anonymous"
        async>
</script>
