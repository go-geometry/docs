---
title: "predicates"
weight: 10
description: "Exact geometric predicates at near floating-point cost."
---

**Status: released, [v0.1.0](https://github.com/go-geometry/predicates/releases/tag/v0.1.0).** Repository [go-geometry/predicates](https://github.com/go-geometry/predicates), wave W1 of the [design](../design/) (tasks P1 to P4).

```sh
go get github.com/go-geometry/predicates@v0.1.0
```

```go
import "github.com/go-geometry/predicates"

predicates.Orient2D(a, b, c)       // > 0 counterclockwise, < 0 clockwise, 0 collinear
predicates.InCircle(a, b, c, d)    // > 0 if d is inside the circle through a, b, c (ccw)
predicates.Orient3D(a, b, c, d)    // > 0 if d is below the plane of a, b, c
predicates.InSphere(a, b, c, d, e) // > 0 if e is inside the sphere through a..d (Orient3D > 0)
predicates.OrientN(points)         // d+1 points in d dimensions, 1 <= d <= 16
predicates.InSphereN(points)       // d+2 points in d dimensions
```

## What it provides

| Function | Method |
|---|---|
| `Orient2D`, `InCircle`, `Orient3D`, `InSphere` | adaptive precision (Shewchuk 1997): a floating-point filter with a proven error bound (stage A), then the exact determinant of the rounded differences (B), a first-order tail correction (C) and the exact determinant (D), each only when the previous stage cannot certify the sign |
| `...Exact` | the exact determinant directly |
| `...Fast` | plain floating-point evaluation, not robust, for comparison |
| `OrientN`, `InSphereN` | exact sign in any dimension 1 ≤ d ≤ 16: floating-point expansion determinants up to order 8, fraction-free big-integer elimination above |

- **Exact sign** for every finite input. Each predicate checks that its coordinates lie in a range, set by the degree of its determinant, in which floating-point expansions cannot overflow or underflow; outside it, an exact big-integer evaluation answers. NaN or ±Inf gives NaN (`OrientN`/`InSphereN` return `ErrNotFinite`).
- **Identical bits on every architecture**: every product is rounded by an explicit `float64(...)` conversion, so the Go compiler never fuses it into an FMA. A test hashes the bits of 5 600 results and compares the hash on amd64, arm64, riscv64, loong64, ppc64le, s390x, 386 and arm.
- **Bounded cost**: `MaxDim = 16`. With coordinates spread over the whole float64 exponent range, the exact elimination took 40 ms at d = 16, 0.9 s at d = 32 and 56 s at d = 64.

## How it is validated

- The sign of every predicate equals the sign of the determinant computed with `big.Rat` on random, near-degenerate (rounded onto a line, circle, plane or sphere, then nudged by a few ulps), exactly degenerate (collinear and coplanar lattices, Pythagorean circles and spheres, repeated points under inexact translations), Shewchuk's tilted grid, and huge / tiny / mixed-magnitude input: zero mismatches. The non-robust `Fast` evaluation is wrong on hundreds of these cases, so the corpus is hard.
- **Planted breaks**: with the stage A or stage B error bound removed, the same corpus must produce wrong signs, or the test fails.
- Fuzz targets for every predicate check the sign against `big.Rat`; their corpus is committed.
- 100 % statement coverage, `-race`, govulncheck.
- Against Shewchuk's `predicates.c` (cgo, `bench/`): equal signs on every benchmark input.

## References

| Reference | Cite |
|---|---|
| Shewchuk, Adaptive Precision Floating-Point Arithmetic and Fast Robust Geometric Predicates, Discrete Comput. Geom. 18:305-363, 1997 | doi:10.1007/PL00009321 |

## Reference implementation and licence

| Implementation | Licence and use |
|---|---|
| Shewchuk's predicates.c | public domain; the package is written from it and the paper (https://www.cs.cmu.edu/~quake/robust.html) |

The package is BSD-3-Clause.

## Depends on

Nothing outside the Go standard library.
