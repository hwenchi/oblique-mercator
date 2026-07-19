# Oblique Mercator

**Live demo:** https://hwenchi.github.io/mercator-3d/

## Rhetorical Design

### Purpose

For an audience of any familiarity with the Mercator projection, we want to
communicate that a map projection is a choice, not a fact. The familiar
distortions of Canada, Greenland, and Russia are easy to mistake for actual
geographic shape. The insight we are after is that those distortions are not
intrinsic to the earth — they are an artifact of a particular projection — and
the way to see this is to watch them shift to other places as the projection
changes.

### Strategy

**Juxtaposition.** The globe and its Mercator cylinder are rendered
simultaneously and transparently. The purpose is to make the correspondence
between the undistorted shape and the projected shape immediate: the viewer can
trace a continent from the sphere to the cylinder and see exactly where and how
the distortion is introduced, without having to hold two separate images in
memory. This is only possible in an interactive 3D visualization — no static or
2D medium can place both representations in the same space at once.

**Continuous deformation.** The cylinder axis is an interactive parameter,
tilting the projection in real time. The purpose is to make the distortion
visibly shift from one region to another — watching Greenland shrink as the
axis tilts, while distortion grows elsewhere, makes the projection feel like a
variable rather than a fixed fact. We deform within the family of oblique
Mercator projections (varying the axis) rather than morphing between different
kinds of projections for two reasons. First, there is no canonical interpolation
between fundamentally different projection types — what is halfway between
Mercator and stereographic is not well-defined — whereas rotating the axis is a
natural, well-defined path. Second, the goal is not visual spectacle but understanding: the viewer should
be able to see *why* the map looks the way it does, not just watch it change.
Morphing between fundamentally different projection types might produce more
dramatic transformations, but the connection between cause and effect would be
opaque. Varying a single geometric parameter — the axis — keeps that connection
legible. This family is a subspace of all projections
homeomorphic to $`\mathbb{RP}^2`$: the axis is a point on $`S^2`$, but since
$`v`$ and $`-v`$ define the same cylinder, antipodal points are identified.

## Technical Challenges

### 1. Projection formula

A rotation $`T \in \mathrm{SO}(3)`$ parameterizes the family: it defines the
cylinder axis and hence which Mercator projection is displayed. The Mercator
formula is only defined for the standard (vertical) axis. Rather than
re-deriving it for an arbitrary axis, we rotate into the cylinder's frame,
apply the standard formula, and rotate back. This gives the projection as a
composition:

```math
P_T = E \circ T \circ M^{-1} \circ T^{-1}
```

where $`E`$ is the equirectangular map and $`M^{-1}`$ is the standard Mercator
inverse, both for the vertical axis:

```math
E(x, y, z) = \bigl(\mathrm{atan2}(y,\, x),\; \mathrm{atan2}(z,\, \sqrt{x^2+y^2})\bigr)
```

```math
M^{-1}(x', y', z') = (\cos\phi'\cos\lambda',\; \cos\phi'\sin\lambda',\; \sin\phi')
```

where $`\lambda' = \mathrm{atan2}(y', x')`$ and $`\phi' = 2\arctan(e^{z'/R}) - \pi/2`$.

The sphere maps to $`(\lambda, \phi)`$ via $`E`$ directly. Since $`M^{-1}`$ is
smooth within $`\pm 85°`$ latitude (where the cylinder is clipped) and $`T`$ is
smooth, $`P_T`$ varies smoothly with $`T`$ — so continuously rotating the axis
continuously deforms the projected image.

### 2. Triangle mesh vs. ray tracing

The sphere and cylinder have exact analytic descriptions. Two rendering
approaches are possible:

| | Triangle mesh | Ray tracing |
|---|---|---|
| **Geometric accuracy** | $`O(h^2)`$ discretization error in face size $`h`$; texture coordinates wrong near poles and at high zoom | Exact up to floating-point rounding |
| **Transparency** | Two render passes: opaque sphere first, translucent cylinder second; depth buffer handles occlusion | Natural — contributions composited in depth order in one pass via Porter-Duff over |
| **Memory** | $`O(N^2)`$ vertices to approximate sphere and cylinder | Constant — geometry is implicit |
| **Complexity** | Tessellation required for curved surfaces | Analytic intersection straightforward for sphere and cylinder |

Ray tracing was chosen primarily for accuracy: the Mercator inverse formula
produces exact geographic coordinates at every pixel regardless of zoom level,
with no discretization parameter to tune.