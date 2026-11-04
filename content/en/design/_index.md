---
title: "Design: scipy.spatial parity"
linkTitle: "Design"
weight: 90
description: "Design of item 7 of the go-scikit plan: the whole of scipy.spatial 1.18.1 plus MATLAB's computational geometry, in pure Go. Inventory, exactness decision per component, proposed repositories, waves and tasks, validation, performance and security."
---

**Status: design, awaiting approval.** No repository exists yet. The repository
names below are proposals for the maintainer to approve. This page replaces
the working names (`hull`, `nearest`) where it disagrees with them.

## 1. Sources opened

Everything below cites only what was opened for this design (2026-11-02/04).

| Source | What it settled |
|---|---|
| SciPy v1.18.1 (`git clone --branch v1.18.1`), `scipy/spatial/` and `subprojects/qhull_r/` | The inventory (section 2), the Qhull options SciPy passes (`_qhull.pyx:1878-1886`, `2486-2496`, `2668-2684`, `2865-2881`), the order of output rows (`get_simplex_facet_array`, `_qhull.pyx:564-713`), the k-d tree build (`ckdtree/src/build.cxx`) and query heap (`query.cxx:22-98`), the distance kernels (`src/distance_metrics.h:49-170`) and the metric registry (`distance.py:1659-1804`) |
| `subprojects/qhull_r/libqhull_r/COPYING.txt` (read before any Qhull code) | The licence verdict (section 3) |
| `libqhull_r/Announce.txt` | Vendored version: Qhull 2020.2 (8.0.2); the reference and abstract of Barber–Dobkin–Huhdanpaa 1996 |
| `libqhull_r/user_r.h:286-315`, `global_r.c:1734-1760` | `qh_RANDOMtype 5` = Qhull's own `qh_rand`; default seed 1: `QJ` is deterministic and portable |
| `scipy/spatial/qhull_misc_README.txt` | SciPy's patch to `qh_new_qhull` (`feaspoint` for halfspaces) |
| Qhull option documentation, `html/qh-optq.htm` (github.com/qhull/qhull) | Meaning of `Qbb Qc Qz Qx Q12 QJ Qt Qu Q11 QbB QR` |
| SPDX licence list, entry `Qhull` | `Qhull` is an SPDX identifier |
| Shewchuk, *Adaptive Precision Floating-Point Arithmetic and Fast Robust Geometric Predicates*, DCG 18(3):305-363, 1997 (CMU-CS-96-140R PDF, pp. 1-3, 46-47) | Radix-2 exact-rounding assumption; section 5 "Caveats": compilers and extended registers break the expansions |
| `predicates.c` header (cs.cmu.edu) and the Quake "robust" page | Public domain; functions `orient2d/3d`, `incircle`, `insphere` with `fast/exact/slow` variants; `exactinit()` |
| Maneewongvatana–Mount, arXiv cs/9901013 (abstract page) | Sliding midpoint rule, as SciPy cites it |
| Edelsbrunner–Mücke, *Three-dimensional alpha shapes*, arXiv math/9410208 (abstract page) | Alpha shapes as a filtration of the Delaunay triangulation, O(n²) worst case |
| Shoemake, *Animating Rotation with Quaternion Curves*, SIGGRAPH 1985, 19(3):245 (page 1) | Citation of slerp |
| libc++ `LICENSE.TXT` | Apache-2.0 WITH LLVM-exception (section 5.3) |
| MathWorks pages `alphaShape`, `delaunayTriangulation`, `KDTreeSearcher` | The MATLAB delta (section 2.6) |

**Not obtained:** the full text of Barber–Dobkin–Huhdanpaa 1996 (ACM returns
403, the archive mirror is unreachable; only the abstract in `Announce.txt` was
read); Markley 2008 (*Unit quaternion from rotation matrix*, JGCD 31(2)) and
Shuster–Markley 2003 and Bernardes–Viollet 2022 (cited in `_rotation.py`, not
opened); the body of Maneewongvatana–Mount and of Edelsbrunner–Mücke (abstracts
only); Shoemake beyond page 1; the MathWorks pages for `convhulln`, `voronoin`,
`pdist2`, `knnsearch`, `rangesearch`, `ExhaustiveSearcher`. The tasks that need
them (R1, R2, M4, M6) open them first.

## 2. Inventory: what parity means

### 2.1 Qhull-backed (SciPy `_qhull.pyx`, `_spherical_voronoi.py`, `_voronoi.pyx`)

| SciPy | Default Qhull options | Surface to port |
|---|---|---|
| `Delaunay(points, furthest_site, incremental, qhull_options)` | `Qbb Qc Qz Q12` (+`Qx` for ndim ≥ 5); `Qt` always; incremental: `Qc`; furthest site drops `Qz`, adds `Qu` | `points, simplices, neighbors, equations, paraboloid_scale/shift, transform, vertex_to_simplex, convex_hull, coplanar, good, vertex_neighbor_vertices, furthest_site, nsimplex`; `find_simplex(xi, bruteforce, tol)`, `plane_distance`, `lift_points`, `add_points(restart)`, `close` |
| `tsearch(tri, xi)` | — | = `find_simplex` |
| `ConvexHull(points, incremental, qhull_options)` | none (`Qt` required) | `points, vertices, simplices, neighbors, equations, coplanar, good, area, volume, max_bound, min_bound, add_points, close`; option `QG` drives `good` |
| `Voronoi(points, furthest_site, incremental, qhull_options)` | `Qbb Qc Qz` (+`Qx` ndim ≥ 5) | `points, vertices, ridge_points, ridge_vertices, regions, point_region, furthest_site, ridge_dict, add_points` |
| `HalfspaceIntersection(halfspaces, interior_point, incremental, qhull_options)` | none | `halfspaces, interior_point, intersections, dual_points, dual_facets, dual_vertices, dual_equations, dual_area, dual_volume`, `add_halfspaces` |
| `SphericalVoronoi(points, radius, center, threshold)` | via `ConvexHull` | `vertices, regions, sort_vertices_of_regions, calculate_areas` |
| `QhullError` | | an error type with Qhull's message |

### 2.2 k-d tree (SciPy C++ `ckdtree/src`, `_ckdtree.pyx`, `_kdtree.py`)

`KDTree` (leafsize 10) and `cKDTree` (leafsize 16): `data, leafsize, m, n,
maxes, mins, size, indices, boxsize, tree` (node view: `level, split_dim,
split, children, data_points, indices, lesser, greater`); options
`compact_nodes, copy_data, balanced_tree, boxsize` (periodic). Methods:
`query(x, k int|list, eps, p, distance_upper_bound, workers)`,
`query_ball_point(x, r, p, eps, workers, return_sorted, return_length)`,
`query_ball_tree`, `query_pairs(r, p, eps, output_type)`,
`count_neighbors(other, r, p, weights, cumulative)`,
`sparse_distance_matrix(other, max_distance, p, output_type)`. Helpers:
`Rectangle` (`volume, split, min/max_distance_point, min/max_distance_rectangle`),
`minkowski_distance`, `minkowski_distance_p`, `distance_matrix`.

### 2.3 `scipy.spatial.distance`

22 metrics (`distance.py:1659-1804`): `braycurtis, canberra, chebyshev,
cityblock, correlation, cosine, dice, euclidean, hamming, jaccard,
jensenshannon, mahalanobis, minkowski, rogerstanimoto, russellrao, seuclidean,
sokalsneath, sqeuclidean, yule` (+ aliases), weights `w` where accepted,
boolean types. Functions: `pdist, cdist` (string or callable metric, `out=`),
`squareform(force, checks)`, `is_valid_dm, is_valid_y, num_obs_dm, num_obs_y`,
`directed_hausdorff(u, v, rng)`, and every single-pair function.
Two code paths exist and both are ported: the C++ kernels
(`_distance_pybind`) behind `pdist/cdist` for most metrics, the C kernels
(`distance_wrap.c`, through `CDistMetricWrapper`) for `correlation, cosine,
jensenshannon, mahalanobis, seuclidean` (correlation = centring, then the
cosine kernel, `distance.py:127-135`), the C `squareform` conversions, and NumPy expressions behind the single-pair functions.

### 2.4 `scipy.spatial.transform`

`Rotation`: `from_quat(scalar_first)`, `from_matrix(assume_valid)`,
`from_rotvec(degrees)`, `from_euler`, `from_davenport`, `from_mrp`,
`as_quat(canonical, scalar_first)`, `as_matrix`, `as_rotvec`, `as_euler`,
`as_davenport`, `as_mrp`, `concatenate`, `apply(inverse)`, `*`, `**`, `inv`,
`magnitude`, `approx_equal`, `mean(weights)`, `reduce`, `create_group`
(I, O, T, Dn, Cn), `identity`, `random`, `align_vectors(weights,
return_sensitivity)`, indexing, `single`, `shape`, iteration. Two backends:
the Cython one (at most one leading dimension) and the array-API one (N-D
stacks, `_rotation.py:27-37`); both are ported, each where SciPy uses it.
`Slerp`, `RotationSpline`. `RigidTransform`: `from_matrix, from_rotation,
from_translation, from_components, from_exp_coords, from_dual_quat, identity,
concatenate, mean, as_matrix, as_components, as_exp_coords, as_dual_quat`,
`*`, `**`, `inv`, `apply(inverse)`, `rotation`, `translation`, indexing.

### 2.5 Others

`procrustes`, `geometric_slerp(start, end, t, tol)`.

### 2.6 MATLAB delta

`delaunayTriangulation` (2-D/3-D, constrained edges in 2-D, `Constraints`,
`convexHull`, `isInterior`, `voronoiDiagram`) and everything it inherits from
`triangulation` (`barycentricToCartesian, cartesianToBarycentric,
circumcenter, edgeAttachments, edges, faceNormal, featureEdges, freeBoundary,
incenter, isConnected, nearestNeighbor, neighbors, pointLocation, size,
vertexAttachments, vertexNormal`); `alphaShape` (`HoleThreshold`,
`RegionThreshold`, default alpha = `criticalAlpha('all-points')`,
`alphaSpectrum, criticalAlpha, numRegions, inShape, alphaTriangulation,
boundaryFacets, perimeter, area, surfaceArea, volume, nearestNeighbor`);
`convhull/convhulln` (`Simplify`, volume), `delaunay/delaunayn`, `voronoin`,
`inpolygon`, `polyarea`; `KDTreeSearcher` (BucketSize 50; euclidean,
cityblock, minkowski, chebychev), `ExhaustiveSearcher`, `createns`,
`knnsearch` (`K`, `IncludeTies`), `rangesearch`, `pdist2`
(`Smallest`/`Largest`).

### 2.7 Left out, and why

| Left out | Why |
|---|---|
| `delaunay_plot_2d`, `convex_hull_plot_2d`, `voronoi_plot_2d`, alphaShape `plot` | Plotting (matplotlib); excluded by the plan. |
| Array-API dispatch to CuPy/JAX/PyTorch, pickling (`__reduce__`, `__getstate__`) | No Go meaning; Go users get the NumPy-backend results. |
| Qhull output formats (`io_r.c` printing, Geomview), statistics (`stat_r.c`), `rbox`, tracing to files, options `TO`/`TI`/`TR` | SciPy never reads them; file options are a security hazard and are refused. The subset of `io_r.c` that builds Voronoi regions (`qh_order_vertexneighbors`, `qh_markvoronoi`, …) **is** ported. |
| MATLAB `polyshape`, `boundary` | Polygon clipping (Vatti / Greiner–Hormann) is a separate subject with its own bibliography; proposed as a later item, not part of item 7. |

## 3. Qhull licence verdict

`COPYING.txt` (Qhull 2020.2, SPDX `Qhull`) allows copying, modifying and
redistributing under five conditions: (1) keep all copyright notices, (2) ship
the text with every copy *and with every product that includes Qhull*, (3) a
modification notice naming who, when and why, (4) say where the original can be
obtained, (5) no warranty. There is no copyleft: a Go translation is a
permitted modified version, and BSD-licensed code may sit beside it.
**Verdict: porting is allowed.** The obligation in (2) travels to every binary
that links the port, so the port lives in its **own repository and module**
(`go-geometry/qhull`, licence file `COPYING.txt` verbatim, `NOTICE` with the
modification record "translated to Go by the go-geometry authors, <date>, to
reproduce SciPy 1.18.1 output in pure Go"), and only the packages that need
Qhull import it. KDTree, distance, transform and predicates never import it.

## 4. Exactness decision per component

| Component | Decision | Equality target |
|---|---|---|
| Delaunay, ConvexHull, Voronoi, HalfspaceIntersection, SphericalVoronoi | **Exact port of libqhull_r 2020.2** (the subset in section 2.7) and of `_qhull.pyx` | Same `simplices`, same row order, same `neighbors`, `coplanar`, `good`; `equations`, `area`, `volume` bitwise against the Linux x86-64 oracle (section 6). An independent exact-predicate implementation could only promise combinatorial equivalence; for cospherical input it would choose different simplices and every consumer test written against SciPy would break. |
| `QJ` joggle | Port `qh_rand` (Park–Miller, `qh_RANDOMtype 5`) and the seed logic of `global_r.c:1734-1760` | Bitwise: the default seed is 1, not the clock. `QR0`/`QR-1` (clock seed) are accepted but not reproducible, as in SciPy. |
| Delaunay `transform` | Port of `_get_barycentric_transforms` (getrf, gecon, getrs; `rcond < 1000·eps` → NaN) using go-ndarray `linalg` | Bitwise if go-ndarray's `getrf` follows the oracle's LAPACK for n ≤ 64; otherwise tolerance 4 ulp per entry, stated in the release. |
| Exact predicates | **Independent**, from Shewchuk 1997 and the public-domain `predicates.c` | Exact signs (never a tolerance); used to validate everything else and by the MATLAB delta. |
| KDTree / cKDTree | **Exact port** of `ckdtree/src` and `_ckdtree.pyx` (BSD-3-Clause), including SciPy's own heap (`query.cxx:32-98`) | Distances bitwise. Indices identical, including ties, **if** the tree layout matches; the layout depends on `std::partition` and `std::nth_element` (`build.cxx:103-121`), which differ between libc++ and libstdc++. Decision: port libc++'s two algorithms (Apache-2.0 WITH LLVM-exception, compatible), giving layout equality with libc++-built wheels (task K0 verifies which wheels those are). libstdc++ (GPL-3.0 with runtime exception) is not ported: against GCC-built wheels the result is equal up to the order of exactly tied neighbours, and weighted `count_neighbors` sums agree to tolerance (their summation order follows the layout). |
| distance (`pdist`, `cdist`) | **Exact port** of the C++ and C kernels | Bitwise. The kernels reduce each pair left to right over the features and interleave 4 pairs (2 when weighted) for ILP (`distance_metrics.h:49-170`): SIMD goes **across pairs**, never across features, without FMA, so parity holds. |
| distance single-pair functions, `directed_hausdorff` | Port of the NumPy expressions over go-ndarray; `directed_hausdorff` shuffles with go-ndarray `random` (MT19937 `RandomState`, PCG64 `Generator`, both present) | Bitwise where go-ndarray is bitwise with NumPy; `norm` follows go-ndarray. |
| `Rotation`, `Slerp`, `RigidTransform`, `geometric_slerp`, `procrustes` | **Exact port** of both backends | Bitwise for elementwise operations (no FMA: every product that Go could fuse is wrapped in an explicit `float64(...)` conversion, the rule of the Go spec). `align_vectors`, `procrustes` and `mean` go through go-ndarray `SVD`/`Eigh`: parity is whatever that library guarantees, stated per release. |
| `RotationSpline` | Exact port; needs `scipy.linalg.solve_banded` | Blocked on the go-scikit linear-algebra item providing `solve_banded`; scheduled last in its wave. |
| MATLAB delta | **Independent implementation** on the exact predicates (MATLAB is closed source) | Combinatorial equivalence and invariants against MATLAB when available and Octave otherwise; `delaunayn/convhulln/voronoin` go through the Qhull port with MATLAB's default options (to be read from MathWorks in task M5). |

**Floating-point rule for the whole organization.** The Go compiler may fuse
`x*y + z` into an FMA on arm64, ppc64le, s390x and riscv64. Every arithmetic
expression in a ported kernel and in the predicates wraps products in
`float64(...)` so that the result does not depend on the architecture; a test per
package runs the kernels on `GOARCH=arm64` and `amd64` and compares bits.
Whether SciPy's own arm64 builds contract FMA (clang's default
`-ffp-contract=on`) is **not verified**: task Q0 measures it. Until then the
bitwise oracle is the Linux x86-64 wheel (baseline x86-64 has no FMA), and other
platforms are compared combinatorially and to tolerance.

## 5. Proposed repositories (for approval)

| Repository | Licence | Content | Imports |
|---|---|---|---|
| `go-geometry/predicates` | BSD-3-Clause (source is public domain) | Expansion arithmetic; `Orient2D/3D`, `InCircle`, `InSphere` (adaptive, exact, fast); exact n-D orientation and in-sphere by expansion determinants | — |
| `go-geometry/qhull` | `Qhull` (separate module) | Port of libqhull_r 2020.2: option parser, Quickhull, merging, Delaunay/Voronoi/halfspace modes, joggle, incremental | — |
| `go-geometry/kdtree` | BSD-3-Clause | Port of cKDTree; replaces the working name `nearest` | go-ndarray |
| `go-geometry/distance` | BSD-3-Clause | Metrics, `pdist/cdist/squareform`, validity helpers, `directed_hausdorff` | go-ndarray, go-simd |
| `go-geometry/transform` | BSD-3-Clause | `Rotation`, `Slerp`, `RotationSpline`, `RigidTransform` | go-ndarray |
| `go-geometry/spatial` | BSD-3-Clause, links `qhull` | The `scipy.spatial` facade: `Delaunay, ConvexHull, Voronoi, HalfspaceIntersection, SphericalVoronoi, Tsearch, Procrustes, GeometricSlerp`; re-exports kdtree and distance | qhull, kdtree, distance, predicates (tests), go-ndarray |
| `go-geometry/triangulation` | BSD-3-Clause | The MATLAB delta: `Triangulation`, `DelaunayTriangulation` (incl. 2-D constrained, shared with item 13), `AlphaShape`, searchers, `pdist2`, `inpolygon`, `polyarea`; replaces the working name `hull` | predicates, kdtree, distance, qhull (for `convhulln/voronoin`) |

Each repository: CLI-free library, 100 % statement coverage, fuzz corpus
committed, govulncheck in CI, README + docs page + landing entry on each
user-visible release.

## 6. Validation plan

- **Oracle.** SciPy 1.18.1 wheels pinned by hash, driven from a Go program that
  generates inputs, runs the oracle and writes JSON fixtures (the oracle never
  runs from shell). Bitwise oracle: `manylinux_2_28_x86_64`; second oracle:
  `macosx_arm64`. Octave (and MATLAB when available) for the delta.
- **SciPy's own tests ported**: `test_qhull.py`, `test_kdtree.py`,
  `test_distance.py` (with the `tests/data/*.txt` matrices, BSD),
  `test_hausdorff.py`, `test_spherical_voronoi.py`, `test_slerp.py`,
  `test__procrustes.py`, `transform/tests/*`. `degenerate_pointset.npz` and
  `selfdual-4d-polytope.txt` are part of the corpus.
- **Equality** as section 4 states it, per component; differences are listed by
  name, never summarised as a count.
- **Invariants checked with exact predicates on every output**: Euler
  characteristic, every Delaunay simplex positively oriented and with an empty
  circumsphere (except where Qhull reports `coplanar`), hull convexity,
  neighbour symmetry, Voronoi duality, k-NN results equal to brute force
  (as multisets of distances).
- **Degenerate corpora**: duplicates, collinear / coplanar / cospherical
  sets, integer lattices, nearly-degenerate tilted lattices (Shewchuk's
  "tilted grid"), points at ±1e±300, mixed scales, `QJ` restarts.

## 7. Performance plan

Single core against SciPy 1.18.1 (`workers=1`, `OPENBLAS_NUM_THREADS=1`),
interleaved rounds, never at 1-min load above 8, allocation counts reported.

| Table | Sizes |
|---|---|
| Delaunay 2-D, 3-D; ConvexHull 3-D, 6-D; Voronoi 2-D | n = 10³, 10⁴, 10⁵, 10⁶ (10⁵ in 6-D) |
| KDTree build, `query` k = 1, 16, `query_ball_point`, `count_neighbors` | n = 10⁶, d = 3; n = 10⁵, d = 16 |
| `cdist`/`pdist` per metric | 1000 × 1000, d = 3, 64, 1024 |
| Rotation `apply`, `as_euler`, `from_matrix`, `*` | 10⁶ rotations |

Order: port exactly, measure, then optimise where the table shows a gap,
algorithm first. Qhull's `mem_r.c` free lists become per-run slabs, `qset_r`
sets become slices with the same element order. distance SIMD kernels go into
go-simd (the lowest shared library), vectorised across pairs. Slower than SciPy
is an open defect.

## 8. Security and fuzzing

- **Untrusted input**: NaN refused (as SciPy); ±Inf refused too, a deliberate divergence
  (SciPy checks only NaN, `_qhull.pyx:284`, and hands infinities to Qhull).
- **Option strings**: the Qhull option parser is fuzzed; file and trace options
  (`TO`, `TI`, `TR`, `Tn` output to a file) are refused.
- **Resource bounds**: Delaunay output grows as n^⌈d/2⌉; every Qhull run takes
  a facet budget and a `context.Context` and returns a refusal, never an
  out-of-memory. Dimension limit 64 (`NPY_MAXDIMS` in `_qhull.pyx`), checked
  before allocation. KDTree recursion depth is bounded (the non-compact
  degenerate path re-enters `build`), `leafsize ≥ 1`, `k` ≤ n.
- **32-bit overflow**: Qhull ids are `int`; SciPy stores indices as `intc`.
  Counts above 2³¹−1 are refused; `pdist` length n(n−1)/2 is computed with
  checked multiplication; `squareform` validates sizes before allocating.
- **Bomb tests** assert a refusal under `GOMEMLIMIT` and `ulimit -v`.
- **Fuzz targets**: option parser, every constructor (points as raw bytes →
  float64), `find_simplex`, KDTree build+query, `from_euler` sequences,
  `squareform`, predicates (sign against `math/big` exact evaluation).

## 9. Waves and tasks (one pull request each)

**W0, foundations**
1. T0.1 go-ndarray `linalg`: export `getrf/getrs`, add `gecon` + `lacn2` (LAPACK `dgecon` parity).
2. T0.2 Oracle harness (Go): pinned SciPy 1.18.1 wheels, fixture writer, JSON schema.
3. Q0/K0 Measurement: FMA contraction in SciPy's arm64 build; libc++ vs libstdc++ in each wheel; `tree.indices` equality on 100 random sets.

**W1, `predicates`**
4. P1 Expansion arithmetic, `Orient2D`, `InCircle` (adaptive stages A–D).
5. P2 `Orient3D`, `InSphere`.
6. P3 Exact n-D orientation and in-sphere.
7. P4 Fuzz against `math/big`, benchmark against `predicates.c`, cross-arch bit test.

**W2, `distance`**
8. D1 Minkowski family kernels (euclidean, sqeuclidean, cityblock, chebyshev, minkowski, weighted).
9. D2 braycurtis, canberra and the boolean metrics.
10. D3 correlation, cosine, seuclidean, mahalanobis, jensenshannon (C kernels).
11. D4 `pdist`, `cdist`, callables, `out=`, `squareform`, `is_valid_*`, `num_obs_*`.
12. D5 Single-pair functions and `directed_hausdorff`.
13. D6 go-simd kernels across pairs; performance table.

**W3, `kdtree`**
14. K1 Build: sliding midpoint, median, compact, periodic `boxsize`, libc++ partition/nth_element, node view.
15. K2 `query` (k list, eps, p, upper bound, workers).
16. K3 `query_ball_point`, `query_ball_tree`, `query_pairs`.
17. K4 `count_neighbors`, `sparse_distance_matrix` (all output types).
18. K5 `Rectangle`, `minkowski_distance(_p)`, `distance_matrix`; fuzz; performance table.

**W4, `qhull`**
19. Q1 Sets, memory, random, option parser (refusals).
20. Q2 Geometry (`geom_r`, `geom2_r`).
21. Q3 Polyhedra and the Quickhull core (`poly_r`, `poly2_r`, `libqhull_r`), `Qt`.
22. Q4 Merging (`merge_r`): `Qx`, `Q12`, `Qc`, coplanar points.
23. Q5 Delaunay mode: `d`, `Qbb`, `Qz`, `Qu`.
24. Q6 Voronoi mode (`v`, the `io_r` subset).
25. Q7 Halfspace mode (`H`, SciPy's `feaspoint` patch).
26. Q8 Joggle `QJ` and restarts.
27. Q9 Incremental mode (`add_points`, `restart`, `Q11`); fuzz; bomb tests.

**W5, `spatial`**
28. S1 `ConvexHull`.
29. S2 `Delaunay` and `Tsearch`.
30. S3 `Voronoi`.
31. S4 `HalfspaceIntersection`.
32. S5 `SphericalVoronoi`.
33. S6 `Procrustes`, `GeometricSlerp`; performance table; v0.1.0 of the facade.

**W6, `transform`**
34. R1 `Rotation` core (quat, matrix per Markley 2008, rotvec, mrp, apply, compose, inverse, magnitude, canonical).
35. R2 Euler and Davenport angles.
36. R3 `mean`, `reduce`, `approx_equal`, `align_vectors`, `random`, `create_group`, `concatenate`, power.
37. R4 N-D array-API backend.
38. R5 `RigidTransform`.
39. R6 `Slerp`, `RotationSpline` (after `solve_banded`).

**W7, `triangulation` (MATLAB delta)**
40. M1 `Triangulation` and its queries.
41. M2 `DelaunayTriangulation` 2-D/3-D (incremental insertion on exact predicates), `isInterior`, `convexHull`, `voronoiDiagram`.
42. M3 2-D constrained edges (shared with item 13).
43. M4 `AlphaShape` 2-D/3-D (after opening the full Edelsbrunner–Mücke paper).
44. M5 `convhull/convhulln` (`Simplify`), `delaunay/delaunayn`, `voronoin` via `qhull`; `inpolygon`, `polyarea`.
45. M6 `KDTreeSearcher`, `ExhaustiveSearcher`, `createns`, `knnsearch` (`IncludeTies`), `rangesearch`, `pdist2` (`Smallest`/`Largest`).

Waves W1, W2, W3 and W6 are independent and may run in parallel; W4 needs
nothing but itself; W5 needs W4 and W3; W7 needs W1 and W3. Consumers: go-approx
scattered interpolation and RBF (W5 S2, W3), optimize `shgo` (W5 S2, S1),
stats QMC Lloyd (W5 S3, W3), item 13 meshing (W1, W7 M3).
