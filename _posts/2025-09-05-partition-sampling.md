---
title: 'Sampling A Random Partition'
date: 2025-09-05
permalink: /posts/2025/09/subset-sampling/
tags:
  - Stirling numbers
  - subsets
  - random partition
  - non-uniform random partition generation
---

# Algorithm Notes

The book

<a href="https://luc.devroye.org/chapter_thirteen.pdf"
aria-label="Click to read the pdf of the
13th chapter of Luc Devroye's non-uniform random variate generation book">Non-Uniform Random Variate Generation</a>
is a classic reference for generating random variables and random objects for simulations and testing purposes.

In Chapter 13 there is a nice application of Accept-Reject sampling to
random partitions. It uses the Stirling numbers of the second kind and can
be leveraged to provide a custom initialization for clustering methods to sample the myriad possible partition initializations.

Once we develop the algorithm in the code snippets we provide an example
at the end where you can use this in a clustering algorithm in scikit-learn

## Key Points

1. A partition of a set is a collection of non-empty subsets where no subset overlaps with any other.
2. There are common algorithms to determine a partition given a set of data. These are often
called unsupervised learning methods.
3. The presence in a non-empty subset means that
the elements in that subset are in the same 'class'.

# Basic Idea of Random Subset Sampling

You've got a non-empty set of items and you want them to be in a partition given a set of data,

$$ \Omega = \{x_1, x_2, \dots, x_n \} $$.

For each finite \\( n \\) value we have a number of partitions with \\( k \\) non-empty subsets.
Not all subsets will be the same size but we will always have \\( k \\) distinct ones.

Now the question of how many \\( k \\) non-empty subsets \\( n \\) objects are there?

These are counted by the Stirling numbers of the second kind. To use enumerate these we use the
implementation in `scipy.special`.

To sample these partitions we use an Accept-Reject scheme. There is some initial, up front computation that must be done. For the numbers, we need to fill out the whole Stirling triangle up to \\( n \\). This is done concisely by leveraging broadcasting.

The Strling numbers themselves get large quickly so for implementation sake we'll keep \\( n \\) small.

# Codewords

Given a finite number of elements we need to be able to map these elements to a given non-empty subset. To represent this mapping concisely we
use strings separated by commas-but there are other ways this could be done too.

For example with \\( n=10 \\) and \\(k=5 \\), the string:

`1,2,3,1,4,3,3,1,1,5`

indicates the first non-empty subset contains elements at indices 0, 3, 7, 8. The second non-empty subset contains the sole element at index 1, the third contains three items, those of indices 2, 5, 6. The fourth contains the sole item at index 4, and the fifth the sole item at index 9. We need a comma delimiter for when \\( k > 9 \\) and \\( n > 10 \\) because otherwise we would be unable to distinguish certain digits.

Here is some source code you can use to generate similar random partitions of 10 items into 5 non-empty subsets. They won't all be the same as the one above.

<details>
  <summary>
        A small-ish example of using Stirling numbers to generate a random
        partition uniformly across all partitions with k non-empty subsets.
  </summary>

```python
import numpy as np
from scipy.special import stirling2


# also note that the default is `exact=False`
# so instead let's
# make it exact=True

n = 10
N = np.array([[i] for i in range(1, n+1)])

# build out Stirling triangle
triangle = stirling2(N, np.array(list(range(1, n+1))), exact=True)

# looks good!
print(triangle)

# alright now let's do the subset sampler
# using the triangle
# we choose n,k = 10, 5, e.g. the numerals 1,...,10 into 5 non-empty subsets
# we will reduce these values to 0 so make a copy if you need the original

n, k = 10, 5

# start with a singleton set and work backwords, the assignments are
# in reverse order
assignments = [k]
# handle the offsets correctly
n -= 1
k -= 1
while n > 0:
    u = np.random.uniform(0, 1)
    if u < triangle[n-1][k-1] / triangle[n][k]:
        assignments.append(k)
        k -= 1
    else:
        assignments.append(np.random.randint(1, k))
    n -= 1

# make it so the 0-index array corresponds to item subset assignment
assignments.reverse()

# now the codeword is assignments
print(assignments)
# handle this in a way that makes passing around easy
# we want each set indication separated by a comma
# this allows the disambiguation of `# nonempty subsets > 9`
# assuming a base 10 representation
# ok now do some stuff
codeword = ','.join(str(val) for val in assignments)
```

</details>

Note that if you want to generate several random samples of partitions you can reuse the same Stirling triangle, it's a one time computational cost for a given \\( n \\).

# Stress Test The Sampler

We can use the `scipy.special` implementation for `stirling2` and work backwords from the last element in the list, now we'll use larger \\( n \\)
and \\( k \\) values though.

<details>
  <summary>
        Scale the arguments of the algorithm up a bit. 
        Use `n=1000` and  `k=50` and sample some partitions.
</summary>

```python3
# let's stress test this one, bigger n,k values
n, k = 1000, 50
# we need to generate the bigger triangle
N = np.array([[i] for i in range(1, n+1)])
triangle = stirling2(N, np.array(list(range(1, n+1))), exact=True)
# note: `triangle` only takes a second or two on my lapto to pre-generate

assignments = [k]
# handle the offsets correctly
n -= 1
k -= 1
while n > 0:
    u = np.random.uniform(0, 1)
    if u < triangle[n-1][k-1] / triangle[n][k]:
        assignments.append(k)
        k -= 1
    else:
        assignments.append(np.random.randint(1, k))
    n -= 1

# again the codeword is assignments
assignments.reverse()
print(assignments)
codeword = ','.join(str(val) for val in assignments)
```

</details>

The above code runs in under a couple of seconds on my macbook air so it's pretty fast. Most of the
cost is the one time setup of the Stirling triangle. If you look at the string stored in `codeword` it will be clear now why we needed the commas in the codeword string.

<details>
  <summary>
       Code to put the random partition generation into
       a function for reusability.
</summary>

```python
def random_partition(n, k, number_samples):
    ## one time costs
    N = np.array([[i] for i in range(1, n+1)])
    triangle = stirling2(N, np.array(list(range(1, n+1))), exact=True)
    samples = []
    original_n_k = (n, k) # these are destroyed on the first run...
    for i in range(number_samples):
        _n, _k = n, k
        assignments = [_k]
        # handle the offsets correctly
        _n -= 1
        _k -= 1
        while _n > 0:
            u = np.random.uniform(0, 1)
            if u < triangle[_n-1][_k-1] / triangle[_n][_k]:
                assignments.append(_k)
                _k -= 1
            else:
                if _k > 1:
                    assignments.append(np.random.randint(1, _k))
                else:
                    assignments.append(1)
            _n -= 1
        assignments.reverse()
        codeword = ','.join(str(val) for val in assignments)
        samples.append(codeword)
    return samples

# this line takes about 25-30 seconds on my machine
random_partition(1000, 50, 10000)
```

</details>

So this is pretty handy but how long does it take?
In practice the cost is usually a second or three to setup the triangle and then very fast to generate the samples. I setup a small bit of code to time things experimentally.

The code to generate the timings:

<details>
  <summary>
   Timeit timing code for the larger
   random partition sample example.
  </summary>

```python
setup_code = """
import numpy as np
from scipy.special import stirling2

def random_partition(n, k, number_samples):
    ## one time costs
    N = np.array([[i] for i in range(1, n+1)])
    triangle = stirling2(N, np.array(list(range(1, n+1))), exact=True)
    samples = []
    original_n_k = (n, k) # these are destroyed on the first run...
    for i in range(number_samples):
        _n, _k = n, k
        assignments = [_k]
        # handle the offsets correctly
        _n -= 1
        _k -= 1
        while _n > 0:
            u = np.random.uniform(0, 1)
            if u < triangle[_n-1][_k-1] / triangle[_n][_k]:
                assignments.append(_k)
                _k -= 1
            else:
                if _k > 1:
                    assignments.append(np.random.randint(1, _k))
                else:
                    assignments.append(1)
            _n -= 1
        assignments.reverse()
        codeword = ','.join(str(val) for val in assignments)
        samples.append(codeword)
    return samples
"""

import timeit
time_str = timeit.timeit(stmt="random_partition(1000, 50, 10000)", setup=setup_code, number=10)
print(f"Time for string join (string stmt): {time_str:.6f} seconds")
```

</details>

empirically for \\( n=1000 \\) and \\( k=50 \\) to generate 10000 random partitions takes approximately 25 seconds. There are many many more partitions than the 10,000 sampled but this gives you a way to see some of them and use them for downstream computations.

One example of this is to start with different initial partitions of some data in a clustering algorithm and determine how sensitive the end result is to the choice of initial partition.

Not all clustering algorithms support custom initializations. One example that does is `KMeans` in scikit-learn. The `init` argument will take a callable input. The implementation of something like this is shown below:

<details>
  <summary>
    Example use in a clustering initialization. Use the technique to test
    the sensitivity of the clustering method to the initialization.
  </summary>

```python
## example custom init using KMeans.
from collections import Counter

import numpy as np
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs

# 0. Define the custom initialization function
def stirling_init(X, n_clusters, random_state):
    ## note this implementation sets up the triangle
    ## for each invocation simply to stay close to the
    ## previous implementations in practice you'd want
    ## something using a closure with the triangle in
    #3 the enclosing environment to prevent the recompute
    n = X.shape[0]
    N = np.array([[i] for i in range(1, n+1)])
    triangle = stirling2(N, np.array(list(range(1, n+1))), exact=True)
    _n, _k = n, n_clusters
    assignments = [_k]
    # handle the offsets correctly
    _n -= 1
    _k -= 1
    while _n > 0:
        u = np.random.uniform(0, 1)
        if u < triangle[_n-1][_k-1] / triangle[_n][_k]:
            assignments.append(_k)
            _k -= 1
        else:
            if _k > 1:
                assignments.append(np.random.randint(1, _k))
            else:
                assignments.append(1)
        _n -= 1
    assignments.reverse()
    ## now use assignments to calculate means in each non-empty partition
    cnts = Counter(assignments)
    X_init = np.zeros((n_clusters, X.shape[1]))
    for idx, i in enumerate(assignments):
        X_init[i-1,:] += X[idx, :]
    for i, cnt in cnts.items():
        X_init[i-1] /= cnt
    return X_init

# test this function out if you want to grok it
x_init = stirling_init(X, n_clusters=n_clusters, random_state=None)


# 1. Create a sample dataset
X, y = make_blobs(n_samples=300, centers=4, n_features=2, random_state=42)

# 2. Instantiate KMeans with the custom callable
kmeans = KMeans(
    n_clusters=4,
    init=stirling_init,  # Pass the function directly
    n_init=1,         # Required to run the algorithm with this custom init
    random_state=42   # For reproducibility
)

# 3. Fit the KMeans model
kmeans.fit(X)

# 4. Check the results-see that they make reasonable sense.
print("Initial centroids (from custom_init):")
print(X[:4])
print("\nFinal cluster centers (from KMeans):")
print(kmeans.cluster_centers_)
```

</details>

IF you rerun the `kmeans.fit(X)` call using the same `X` input a second time and inspect the cluster centers across multiple runs you may notice cycling
behavior where the vectors are the same set of 4 rows but the values of the rows of the cluster center matrix are permutated.datasets

This is a well know behavior in KMeans and is something to account for if you are writing Markov Chain Monte Carlo
algorithms. See the book by Casella and  Robert for more details.

To speed this up some you'd probably want to precompute a collection of the partition codewords and then iterate through them as inputs to generate initial values passed into a custom initialization function. The way I've done
it here in the post is for easier reading and understanding not necessarily for efficient execution.
