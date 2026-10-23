---
title: "nearest"
weight: 30
description: "k-d trees and exact nearest-neighbour search."
---

<!-- From go-scikit/roadmap PLAN.md and bib/geometry-pde-graphs.md (section 2, algorithms; section 3, implementations; section 4, validation). The package has no repository yet: change the roadmap first, then here. -->

**Status: planned (wave 2).** There is no repository and no code yet, and `nearest` is a working name. These pages describe the plan, not a release.

k-d trees and exact nearest-neighbour search.

## What it will provide

| | |
|---|---|
| search | exact nearest neighbours and k nearest neighbours in low dimension |
| facade | a MATLAB-style `knnsearch` API over go-ndarray |

## References

The algorithms and the theory the package will start from, as the roadmap's bibliography lists them:

| Reference | Cite |
|---|---|
| Bentley, Multidimensional binary search trees used for associative searching, CACM 18(9):509-517, 1975 | doi:10.1145/361002.361007 |

## Reference implementations and licences

What exists, under which licence, and what that allows:

| Implementation | Licence and use |
|---|---|
| gonum/gonum `spatial/kdtree` and `vptree` | BSD-3-Clause; to be adopted behind the facade, not rewritten (https://github.com/gonum/gonum) |
| SciPy `cKDTree` | BSD-3-Clause; the oracle |

GPL, LGPL, AGPL and non-free code is never ported: it is only run as an oracle, as an external program.

## How it will be validated

- Index sets exactly equal to a brute-force search, ties broken by index, and to the oracle's.

## Depends on

gonum and go-ndarray.

## Where the plan lives

[PLAN.md](https://github.com/go-scikit/roadmap/blob/main/PLAN.md), [STATUS.md](https://github.com/go-scikit/roadmap/blob/main/STATUS.md) and the [bibliography](https://github.com/go-scikit/roadmap/blob/main/bib/geometry-pde-graphs.md) in the go-scikit roadmap.
