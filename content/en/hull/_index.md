---
title: "hull"
weight: 20
description: "Convex hulls, Delaunay triangulations, Voronoi diagrams and alpha shapes."
---

<!-- From go-scikit/roadmap PLAN.md and bib/geometry-pde-graphs.md (section 2, algorithms; section 3, implementations; section 4, validation). The package has no repository yet: change the roadmap first, then here. -->

**Status: planned (wave 2).** There is no repository and no code yet, and `hull` is a working name. These pages describe the plan, not a release.

Convex hulls, Delaunay triangulations, Voronoi diagrams and alpha shapes.

## What it will provide

| | |
|---|---|
| convex hull | 2D, 3D and n-dimensional (Quickhull, with merged facets for degenerate input) |
| Delaunay | 2D and 3D triangulations |
| Voronoi | as the dual of the Delaunay triangulation |
| alpha shapes | a filtration of the Delaunay triangulation |

## References

The algorithms and the theory the package will start from, as the roadmap's bibliography lists them:

| Reference | Cite |
|---|---|
| Barber, Dobkin, Huhdanpaa, The Quickhull algorithm for convex hulls, ACM TOMS 22(4):469-483, 1996 | doi:10.1145/235815.235821 |
| Guibas, Stolfi, Primitives for the manipulation of general subdivisions and the computation of Voronoi diagrams, ACM TOG 4(2):74-123, 1985 | doi:10.1145/282918.282923 |
| Fortune, A sweepline algorithm for Voronoi diagrams, Algorithmica 2:153-174, 1987 | doi:10.1007/BF01840357 |
| Edelsbrunner, Mücke, Three-dimensional alpha shapes, ACM TOG 13(1):43-72, 1994 | doi:10.1145/174462.156635 |

## Reference implementations and licences

What exists, under which licence, and what that allows:

| Implementation | Licence and use |
|---|---|
| SciPy `spatial` | BSD-3-Clause; the oracle for hulls, Delaunay and Voronoi (it wraps Qhull) |
| Qhull | custom licence with five conditions, not an SPDX one: an oracle until those conditions are read |
| Octave `delaunay`, `convhull` | an oracle that mirrors MATLAB semantics |
| CGAL | GPL-3.0+ or LGPL-3.0+: an oracle only |

GPL, LGPL, AGPL and non-free code is never ported: it is only run as an oracle, as an external program.

## How it will be validated

- Combinatorial validity: the Euler relation, and the empty-circumsphere test with exact predicates.
- The vertex set equals the oracle's up to degeneracies; the number of differing simplices is reported only for cocircular input, where non-uniqueness is legitimate.

## Depends on

`predicates`, and go-ndarray for the array facade.

## Where the plan lives

[PLAN.md](https://github.com/go-scikit/roadmap/blob/main/PLAN.md), [STATUS.md](https://github.com/go-scikit/roadmap/blob/main/STATUS.md) and the [bibliography](https://github.com/go-scikit/roadmap/blob/main/bib/geometry-pde-graphs.md) in the go-scikit roadmap.
