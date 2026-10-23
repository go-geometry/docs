---
title: "predicates"
weight: 10
description: "Exact geometric predicates at near floating-point cost."
---

<!-- From go-scikit/roadmap PLAN.md and bib/geometry-pde-graphs.md (section 2, algorithms; section 3, implementations; section 4, validation). The package has no repository yet: change the roadmap first, then here. -->

**Status: planned (wave 2).** There is no repository and no code yet, and `predicates` is a working name. These pages describe the plan, not a release.

Exact geometric predicates at near floating-point cost.

## What it will provide

| | |
|---|---|
| 2D | `Orient2D`, `InCircle` |
| 3D | `Orient3D`, `InSphere` |
| method | adaptive precision: a floating-point filter first, exact expansion arithmetic only when the filter cannot decide the sign |

## References

The algorithms and the theory the package will start from, as the roadmap's bibliography lists them:

| Reference | Cite |
|---|---|
| Shewchuk, Adaptive Precision Floating-Point Arithmetic and Fast Robust Geometric Predicates, Discrete Comput. Geom. 18:305-363, 1997 | doi:10.1007/PL00009321 |

## Reference implementations and licences

What exists, under which licence, and what that allows:

| Implementation | Licence and use |
|---|---|
| Shewchuk's predicates.c | public domain; may be ported (https://www.cs.cmu.edu/~quake/robust.html) |

GPL, LGPL, AGPL and non-free code is never ported: it is only run as an oracle, as an external program.

## How it will be validated

- The sign of each predicate equals the sign computed with `big.Rat` arithmetic on random and near-degenerate input; zero mismatches.
- Degenerate families (cocircular grids, collinear sets) are generated in the repository.

## Depends on

nothing. Every other package of the organization depends on it.

## Where the plan lives

[PLAN.md](https://github.com/go-scikit/roadmap/blob/main/PLAN.md), [STATUS.md](https://github.com/go-scikit/roadmap/blob/main/STATUS.md) and the [bibliography](https://github.com/go-scikit/roadmap/blob/main/bib/geometry-pde-graphs.md) in the go-scikit roadmap.
