---
title: "Algorithmic Differentiation of QR decompositions"
collection: publications
category: conferences
permalink: /publication/2024-09-17-Algorithmic-Differentiation
excerpt: 'Matrix Derivatives to propagate gradients through all QR matrices.
This paper proves two formula mathematically equivalent for real matrices with
real valued entries and non-equivalence for complex valued matrices. For complex
field entries, we derivate an adjustment that enables library developers and
maintainers to easily add support for complex matrices.
AlgoDiff 2024.'
date: 2024-09-17
venue: 'AlgoDiff 2024'
paperurl: ''
citation: ''
---

Slides from the talk are linked [here](https://rlucas7.github.io/talks/2024-09-17-talk)
These contain some additional context that is not present in the [preprint](https://arxiv.org/abs/2009.10071)
However, some items (flop count derivations) are not presented (for reasons of time).
Flop counts and other updates are in the version to be published by Springer.

What is a QR decomposition and why should you care?
================
A [QR decomposition](https://en.wikipedia.org/wiki/QR_decomposition)
is a fundamental decomposition in matrix analysis.
The decomposition appears as a subroutine in eigenvalue and eigenvector calculations,
solving least squares problems, and many applications with Krylov subspaces.

If you've done any matrix algebra calculations in an application,
you've probably used a QR decomposition and weren't aware.

The Results
========
We demonstrate a new equation for propagating a gradient backwards through a
[DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph)
which has a QR decomposition. We derive the mathematical equivalence between
our formula and an existing formula. We argue that our formula is better and should
therefore be preferred.


The Recommendations
============

Use our formula for the Matrix gradient backpropagation. It is more efficient in
practice and in theory. We show simulation studies with timings and derive flop
counts for comparisons via asymptotic considerations.

Critiques
=========

Some critiques I heard at the AlgoDiff conference:

1. We did not include flop counts in our comparisons of the two formula
2. We needed example matrices for an empirical study
3. Applications were not fleshed out in enough detail

These three critiques are valid-and helpful-we addressed each one in the revised
version which is submitted for publication. A section with flop
count derivations was added. The finding is that our formula has
lower flop counts, therefore is faster in general for larger scale matrices.
The flop count derivations are not in the slides but the empirical results are
on slides 21-23 from the talk.

To address the second comment a subsection with a table was added and I compared
against several matrices from Matrix Market matrix database. This is a standard
collection of matrices to use as test problems and easily facilitates comparisons.
If you're looking for these results they are in slide 25 from the talk.

To address the third comment, potential applications were added. The paper really
is focused on the theory and demonstrating and argument our formula is better rather
than focusing on applications. However, some applications were included in the
revised version. For example, the occurrence of wide matrices was a point that
was raised, some discussants suggested that these matrices are rare. In the revised
version I have some discussion of when they occur. For example any physics application
where the [Buckingham Pi theorem](https://en.wikipedia.org/wiki/Buckingham_%CF%80_theorem#Examples)
is used will have a wide matrix. This theorem applies in many applications and
out results enable you to propogate a matrix derivative through a compute DAG
that leverages the Buckingham Pi theorem. Common physics applications that leverage
the Buckingham Pi theorem are in computational fluid dynamics, amongst many others.

Also, you see wide matrices in a lot of deep learning workflows where you want
to multiply a tall matrix with another to get a less tall-or square-matrix.
Leveraging our results will make these nodes in the DAG much more efficient.

Conclusion
==========

If you are writing a gradient propagation software that supports matrices then
when you develop support for QR decompositions I'd recommend you read our paper
and leverage our findings.

If you are interested in doing further research in this area, feel free to drop
me an email. I have some ideas and am always happy to speak to potential
research partners both for theoretical research (new formula and algorithms) as
well as applications (using QR and backprop in your application).

If you are using PyTorch or TensorFlow, you already have access to these algorithms.
We merged them as part of this work effort.

This work has been used in [several subsequent papers](https://scholar.google.com/scholar?cites=6667333045569434596&as_sdt=5,33&sciodt=0,33&hl=en),
including [a nice paper in NeurIPS 2024](https://proceedings.neurips.cc/paper_files/paper/2024/file/58b286aea34a91a3d33e58af0586fa40-Paper-Conference.pdf).

That looks at scaling *gradients of functions of Matrices*, in these setups you
often use eigevalue and eigenvector calculations as subroutines, and those often use QR.


Updates
=======
Once the submitted paper is published by springer I'll add a link here.
