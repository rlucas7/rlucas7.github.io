---
title: "Generating r-Stirling Numbers of the Second Kind with Python"
date: 2025-09-08
---

In this blog post, we will explore the r-Stirling numbers of the second kind, a generalization of the well-known Stirling numbers of the second kind. We will see how to generate them using a recursive Python function and the `scipy.special` [method](https://docs.scipy.org/doc/scipy/reference/special.html) for stirling numbers of the second kind. We will also discuss how to use and how to create a NumPy ndarray of these numbers.

## What are r-Stirling Numbers of the Second Kind?

The Stirling numbers of the second kind, denoted as $$ S(n,k) $$, count the number of ways to partition a set of $n$ elements into \\( k \\) non-empty subsets.

The r-Stirling numbers of the second kind, denoted as $$ S_r(n,k) $$, add a constraint: they count the number of partitions of a set of \\( n \\) elements into \\( k \\) non-empty subsets, with the condition that the first \\( r \\) elements must be in distinct subsets.

[Andrei Broder](https://en.wikipedia.org/wiki/Andrei_Broder), a distinguished scientist at Google, PhD student of Don Knuth, and Information Retrieval researcher, wrote a paper that describes their properties. These numbers are used in various hashing calculations including in [Bloom filters](https://en.wikipedia.org/wiki/Bloom_filter).

## A Recursive Formula

The r-Stirling numbers of the second kind can be calculated using the following recurrence relation for \\( n > r \\):

$$ S_r(n, k) = k \cdot S_r(n-1, k) + S_r(n-1, k-1) $$

with the following base cases:
- $$ S_r(n, k) = 0 $$ if \\( k < r \\) or \\( n < k \\)
- $$ S_r(r, r) = 1 $$

## Generating an ndarray of r-Stirling Numbers with Cross Recurrence

We can use this function to generate a 2D NumPy array of r-Stirling numbers for a given range of \\( n \\) and \\( k \\) starting with with \\( r=1 \\).

Once we have the \\( r=1 \\) face, to generate r-Stirling numbers is to use a cross recurrence formula from Broder's paper [The r-Stirling Numbers](http://infolab.stanford.edu/pub/cstr/reports/cs/tr/82/949/CS-TR-82-949.pdf). This method allows us to build the table of r-Stirling numbers for a given \\( r \\) from the table for \\( r-1 \\).

The cross recurrence formula is as follows:

$$ S_r(n,k) = S_{r-1}(n, k) - (r - 1) S_{r-1}(n-1, k) $$

We can use this recurrence to build a 3D NumPy array of r-Stirling numbers. We start with the \\( r=1 \\) face, which corresponds to the standard Stirling numbers of the second kind, and then iteratively build the faces for \\( r=2, 3, ... \\). For simplicity of indexing we include a face of all 0 values.

### Python Implementation with Cross Recurrence

Here is a Python function that implements this method using `scipy.special.stirling2` to generate the initial `r=1` face.

```python
import numpy as np
from scipy.special import stirling2

def r_stirling(n_max, k_max, R):
    """
    Generates a 3D NumPy array of r-Stirling numbers up to R.
    """
    array = np.zeros((R + 1, n_max + 1, k_max + 1), dtype=int)

    # Generate r=1 face using scipy.special.stirling2
    N = np.array([[i] for i in range(n_max+1)])
    triangle = stirling2(N, np.array(list(range(k_max+1))), exact=True)
    # build out Stirling triangle
    array[1, :, :] = triangle
    # Generate faces for r = 2 to R using the cross recurrence
    for r in range(2, R + 1):
        for n in range(r, n_max + 1):
            for k in range(1, k_max + 1):
                array[r, n, k] = array[r - 1, n, k]
                array[r, n, k] -= (r - 1) * array[r - 1, n - 1, k]
    return array

# Example usage:
n_max = 8
k_max = 8
R = 3
rs = r_stirling(n_max, k_max, R)

print(f"r-Stirling numbers for r=2:")
print(rs[2])

print(f"\nr-Stirling numbers for r=3:")
print(rs[3])
```

These faces are given in the paper by Broder so we can check the values against Table 1, where we notice the values match up; they do.

Feel free to use this as a starting point for your r-Stirling number of the second kind research. The calculations are very fast and are done in exact arithmetic.

One place where r-Stirling numbers arise is in [Coupon Collecting](https://en.wikipedia.org/wiki/Coupon_collector%27s_problem). This is a problem that arises when you have a set of items, each with an associated probability of being collected.
While the classical solution calculates the expected value of the number of trials to collect all items, there is a way, using the r-Stirling numbers of the second kind, to calculate the probability distribution exactly. The combinatorial calculations are derived in [a recent paper](https://ajc.maths.uq.edu.au/pdf/78/ajc_v78_p376.pdf) by Greg Morrow of the University of Colorado, Colorado Springs.

There are many variants of r-Stirling numbers. The [associated r-Stirling number of the second kind](https://cdm.ucalgary.ca/article/view/68674/54579) enumerate partitions of \\( n \\) elements into non-empty subsets where each subset has at least \\( r \\) items, e.g., if \\( r>1 \\) then there are no isolated/singleton sets. These are popular in practical clustering applications where you do not want isolated clusters, instead you want to have a minimum number of points per cluster. This is a criterion often desired by businesses and other organizations using clustering.