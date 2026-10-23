---
title: "go-geometry documentation"
linkTitle: "Home"
type: docs
cascade:
  type: docs
description: "go-geometry (planned, wave 2): robust predicates, convex hulls, Delaunay triangulations, Voronoi diagrams, k-d trees and mesh generation in pure Go (CGO=0), validated against SciPy, Octave and the Qhull and Triangle binaries."
---

**go-geometry is planned, in wave 2.** It holds no library yet: no repository
contains code, and nothing here is released. These pages say what the
organization will cover, which items of the plan it carries, and which
references each package will be validated against.

## Scope

Robust predicates, convex hulls, Delaunay, Voronoi, k-d trees, mesh generation.

## The plan

In the [go-scikit roadmap](https://github.com/go-scikit/roadmap),
[PLAN.md](https://github.com/go-scikit/roadmap/blob/main/PLAN.md) puts this
organization at item 7 of wave 2 (exact predicates, Quickhull, Delaunay, Voronoi, alpha shapes, k-d trees) and, in wave 3, item 13 (meshing). The plan's rules apply to every item: pure Go (CGO=0) on
every architecture the fleet's CI tests, a bibliography read before any code,
validation against the tools that are the reference in the domain (run as
oracles, never ported when their licence forbids it), unit and regression tests
at 100% coverage, and a performance table against the reference tool.

## Status

[STATUS.md](https://github.com/go-scikit/roadmap/blob/main/STATUS.md) lists
each item of the plan with its repository, pull requests, release and measured
parity. This organization's items are **planned**. The evidence behind the scope
(function inventories, algorithms, reference implementations and licences,
validation oracles) is in the roadmap's
[bibliography](https://github.com/go-scikit/roadmap/blob/main/bib/geometry-pde-graphs.md).

## The planned packages

The package names are working names. None exists as a repository yet.

| Package | Status | What it will provide |
| --- | --- | --- |
| [predicates]({{< relref "predicates" >}}) | planned (wave 2), no repository yet | Exact geometric predicates at near floating-point cost. |
| [hull]({{< relref "hull" >}}) | planned (wave 2), no repository yet | Convex hulls, Delaunay triangulations, Voronoi diagrams and alpha shapes. |
| [nearest]({{< relref "nearest" >}}) | planned (wave 2), no repository yet | k-d trees and exact nearest-neighbour search. |
| [meshgen]({{< relref "meshgen" >}}) | planned (wave 3), no repository yet | Constrained Delaunay triangulation and quality mesh generation, 2D then 3D. |

## Licences

Safe sources to port from are Shewchuk's predicates (public domain), the published papers, gonum (BSD-3-Clause) and MFEM (BSD-3-Clause). Triangle, TetGen, CGAL and Gmsh are never ported: they are run as external oracles. Qhull's licence has five custom conditions that are read before anything is ported from it.

## Neighbouring organizations

go-fem solves PDEs on the meshes of this organization. go-graphalgo is the sibling for graphs. go-approx covers scattered-data interpolation.

## What each package's pages hold

| Package | Pages |
| --- | --- |
| predicates | [overview]({{< relref "predicates" >}}) |
| hull | [overview]({{< relref "hull" >}}) |
| nearest | [overview]({{< relref "nearest" >}}) |
| meshgen | [overview]({{< relref "meshgen" >}}) |
