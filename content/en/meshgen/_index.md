---
title: "meshgen"
weight: 40
description: "Constrained Delaunay triangulation and quality mesh generation, 2D then 3D."
---

<!-- From go-scikit/roadmap PLAN.md and bib/geometry-pde-graphs.md (section 2, algorithms; section 3, implementations; section 4, validation). The package has no repository yet: change the roadmap first, then here. -->

**Status: planned (wave 3).** There is no repository and no code yet, and `meshgen` is a working name. These pages describe the plan, not a release.

Constrained Delaunay triangulation and quality mesh generation, 2D then 3D.

## What it will provide

| | |
|---|---|
| 2D | constrained Delaunay triangulation with Ruppert/Chew refinement and a minimum-angle guarantee |
| 3D | constrained Delaunay tetrahedralisation with refinement, after the 2D work |

## References

The algorithms and the theory the package will start from, as the roadmap's bibliography lists them:

| Reference | Cite |
|---|---|
| Ruppert, A Delaunay refinement algorithm for quality 2-dimensional mesh generation, J. Algorithms 18(3):548-585, 1995 | doi:10.1006/jagm.1995.1021 |
| Shewchuk, Delaunay refinement algorithms for triangular mesh generation, Comput. Geom. 22:21-74, 2002 | doi:10.1016/S0925-7721(01)00047-5 |
| Si, TetGen, a Delaunay-based quality tetrahedral mesh generator, ACM TOMS 41(2):11, 2015 | doi:10.1145/2629697 |
| Geuzaine, Remacle, Gmsh, IJNME 79(11):1309-1331, 2009 | doi:10.1002/nme.2579 |

## Reference implementations and licences

What exists, under which licence, and what that allows:

| Implementation | Licence and use |
|---|---|
| Triangle 1.6 | not free for commercial use: an oracle only, built locally and never vendored |
| TetGen 1.5 | AGPL-3.0-or-later or commercial: an oracle only |
| Gmsh | GPL-2.0-or-later with a linking exception: an oracle, run as an external binary |

GPL, LGPL, AGPL and non-free code is never ported: it is only run as an oracle, as an external program.

## How it will be validated

- Mesh quality: minimum angle, maximum aspect ratio and area conservation, reported as distributions and not only as a minimum.
- A mesh is never compared with an oracle's node by node.

## Depends on

`hull` and `predicates`. go-fem consumes the meshes.

## Where the plan lives

[PLAN.md](https://github.com/go-scikit/roadmap/blob/main/PLAN.md), [STATUS.md](https://github.com/go-scikit/roadmap/blob/main/STATUS.md) and the [bibliography](https://github.com/go-scikit/roadmap/blob/main/bib/geometry-pde-graphs.md) in the go-scikit roadmap.
