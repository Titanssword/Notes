
# Introduction
## two definition
- Single quantile approximation problem
- All quantile approximation problem

## Related Work
    Deterministic algorithm MRL,GK(the best known Deterministic algorithm)
    Allowing randomness enables sampling, we can feed the sampled elements into a GK sketch. However, to produce such samples, one must know n (at least approximately) in advance. Since n is not known in advance it is not a *trival* tast to combine *sampling wich GK sketches.*
### Mergeability
    This property allows one to sketch different sections of the stream independently and then combine the resulting sketches.
    In the paper "Mergeable summaries" Agarwal el conjecture that the GK sketch is not Mergeable
### Model
    different sketching algorithm perform different kinds of operations on the elements in the stream.
    - The most restricted model is the comparison model.
    - Another model assumes that total size of the universe is bounded by |U|


## Main contribution
#### Compactor
    A Compactor can store k items all with the same weight w, it can also compact its k elements into k/2 elements of weight 2w as follows.
    First, the items are sorted. Then, either the even or the odd elements is doubled(set to 2w).

    Figure 1: An illustration of a single Compactor with 6 items performing a single compaction operation. The rank of a query remains unchanged if its rank with in the compactor is even. If it is odd, its rank is increased or decreased by w with equal probability by the compaction operation.

#### First Improvement
    1. to use different compactor capacities in different heights, denoted by kn.
