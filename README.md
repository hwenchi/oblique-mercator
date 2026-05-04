# Mercator 3D

**Live demo:** https://galmungral.github.io/mercator-3d/

## Rhetorical Design

### Goal

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
memory.

**Continuous deformation.** The cylinder axis is an interactive parameter,
tilting the projection in real time. The purpose is to make the distortion
visibly shift from one region to another — watching Greenland shrink as the
axis tilts, while distortion grows elsewhere, makes the projection feel like a
variable rather than a fixed fact. This requires situating Mercator within a
continuous family of projections. The family parameterized by cylinder axis is
a subspace of all projections homeomorphic to $`\mathbb{RP}^2`$: the axis is a
point on $`S^2`$, but since $`v`$ and $`-v`$ define the same cylinder,
antipodal points are identified.

## Technical Challenges

### 1. Projection formula

A rotation $`T \in \mathrm{SO}(3)`$ parameterizes the family: it defines the
cylinder axis and hence which Mercator projection is displayed. Both surfaces
share an equirectangular map in which $`(u, v) \in [0,1]^2`$ encodes longitude
$`\lambda = 2\pi u - \pi`$ and latitude $`\phi = \pi/2 - \pi v`$.

**Sphere.** A point $`\mathbf{p}`$ on the unit sphere gives UV directly:

```math
u = \frac{\operatorname{atan2}(p_y, p_x) + \pi}{2\pi}, \qquad v = \frac{\pi/2 - \operatorname{atan2}(p_z,\, \|\mathbf{p}_{xy}\|)}{\pi}
```

**Cylinder.** A point $`\mathbf{p}`$ on the cylinder is brought into the
cylinder's frame by $`\mathbf{p}_0 = T^{-1}\mathbf{p}`$. Its cylinder
coordinates $`(\theta_0, z_0)`$ are inverted through the Mercator formula to
recover the geographic latitude:

```math
\phi_0 = 2\arctan\!\left(e^{z_0/R}\right) - \frac{\pi}{2}
```

The corresponding sphere point $`\mathbf{r}_0 = (\cos\phi_0 \cos\theta_0,\,
\cos\phi_0 \sin\theta_0,\, \sin\phi_0)`$ is rotated back to world space
$`\mathbf{r} = T\mathbf{r}_0`$, from which UV is read using the sphere formula.
Since the Mercator formula is smooth away from the poles and the cylinder is
clipped to $`\pm 85°`$ latitude, the projection varies smoothly with $`T`$ —
so continuously rotating the axis continuously deforms the projected image.

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