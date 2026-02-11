# What is a CRS?

<!-- draft chapter opening, brainstormed 2026-02-11 -->
<!-- status: RAW DRAFT — voice capture, needs tightening and figures -->

Every "map projection" is not just one thing. It's a coordinate reference system — a CRS — and every CRS is composed of parts: a family, a datum, a centre, and other parameters. Too often we equate "a map projection" to a short code like `EPSG:3031` or to a particular parameterized variant like "Polar Stereographic." That collapses several independent choices into one opaque label, and it makes projections seem like magic incantations rather than decisions you can reason about.

Let's unpack this.

## Family

The family is the overall kind of projection. It determines the *nature* of the compromise — what geometric properties are preserved, what's sacrificed, and in what pattern across the map. A family is a name: Lambert Azimuthal Equal Area, Mercator, Albers Conic, Stereographic, Gnomonic. Each name corresponds to a `+proj=` code: `laea`, `merc`, `aea`, `stere`, `gnom`.

<!-- [TABLE or FIGURE: show 4–6 families side by side, same data, same centre, 
     to show what "family" means visually — the *kind* of distortion pattern] -->

Families capture relative properties. A conformal family (Mercator, Stereographic) preserves local angles and shapes at the cost of area. An equal-area family (Lambert Azimuthal, Albers) preserves area at the cost of shape. An equidistant family preserves distance along one axis or from one point. No family can do all three — that's the fundamental compromise, and the family is where you make it.

You've seen the guides that explain this. They show a light source, a globe, and a geometric object — a plane, a cone, a cylinder — wrapped around or touching the globe. These are the *developable surfaces*, and every projection family is built from one of them. The plane gives you azimuthal projections (Stereographic, Lambert Azimuthal, Gnomonic). The cone gives you conic projections (Albers, Lambert Conformal Conic). The cylinder gives you cylindrical projections (Mercator, Equal Area Cylindrical, Plate Carrée).

<!-- [FIGURE: the classic three developable surfaces — but annotated with 
     "this tells you the family, not the map"] -->

These guides tell a nice story, and then you're scratching your head about how to actually understand what a map projection *is*, what it's *for*, and most importantly how to *do it*. Because the developable surface tells you the family of mathematics involved, but it doesn't tell you where the cone sits, or which way the cylinder is oriented, or what part of the Earth the plane is tangent to. For that you need the rest of the CRS.

## Centre

When you actually use a projection, the centre dictates where the middle of the map falls on the Earth. It's the point of least distortion — the place where the developable surface touches or intersects the globe. Move the centre and you move the sweet spot.

In PROJ notation this is `+lat_0` and `+lon_0`. For a south polar map, `+lat_0=-90`. For a map centred on Hobart, `+lat_0=-42.88 +lon_0=147.33`. Same family, completely different map.

<!-- [FIGURE: same family (laea), three different centres — polar, mid-latitude, 
     equatorial — showing how the distortion pattern rotates with the centre] -->

You might think Mercator and longitude-latitude don't have parameterizable centres. They absolutely do. Standard Mercator has `+lon_0=0` by convention, but change it and the map recentres — the antimeridian moves, the seam shifts. More dramatically: rotate Mercator 90 degrees and the equator becomes the central *meridian* instead of the central parallel. That's Transverse Mercator — and UTM is just Transverse Mercator applied in 60 six-degree strips, each with its own `+lon_0`. The Universal Transverse Mercator system is not a single projection. It's one family, sixty centres.

Similarly, longitude-latitude (`+proj=longlat`) has a default centre at (0, 0) — the intersection of the equator and the prime meridian. But Oblique Transverse (`+proj=omerc`), Cassini-Soldner (`+proj=cass`), and Transverse Mercator (`+proj=tmerc`) are all the same family of cylinder-on-the-globe, just oriented differently. The family is cylindrical; the centre and orientation are what make each one a different CRS.

<!-- [FIGURE: Mercator → Transverse Mercator → Oblique Mercator, 
     showing the cylinder rotating around the globe] -->

## Other parameters

Beyond family and centre, a CRS has parameters that control the specific geometry of the projection surface and various practical offsets:

**Latitude of true scale** (`+lat_ts`): for Stereographic and some cylindrical projections, this is where scale is exactly 1:1. NSIDC's south polar stereographic uses `+lat_ts=-70` — scale is perfect at 70°S, distorted elsewhere.

**Standard parallels** (`+lat_1`, `+lat_2`): for conic projections, these are the two latitudes where the cone intersects the globe. Between them, distortion is minimised. A Lambert Conformal Conic for Tasmania might use `+lat_1=-50 +lat_2=-35` — the cone cuts through the region of interest.

**False easting and northing** (`+x_0`, `+y_0`): arbitrary offsets to avoid negative coordinates. Pure convenience — they shift the map in the plane without changing anything about the projection geometry.

**Datum and ellipsoid** (`+datum`, `+ellps`): the shape of the Earth model. WGS84 is the most common, but older datasets use Clarke 1866, GRS80, or the Hughes 1980 ellipsoid (used by NSIDC for northern hemisphere sea ice). The datum affects where coordinates land on the ground — change the datum and the same lon-lat pair points to a different physical location, potentially by hundreds of metres.

## A CRS is not a map

Here's the thing that's easy to miss: a CRS — even a fully parameterized one with family, centre, datum, and all the rest — is not a map. A CRS defines how the curved Earth maps to a flat plane. It says nothing about *which part* of that plane you're looking at.

For a map, you also need a bounding box: the spatial extent in the projected coordinate system. The CRS tells you the rules of the transformation; the extent tells you the window. Two maps can use the exact same CRS and show completely different parts of the world, at completely different scales.

This is why "what projection should I use?" is only half the question. The other half is "what extent do I need?" — and as we'll see, those two questions are coupled. The extent that looks right in one projection looks wrong in another, because the distortion pattern changes.

## Make a map

So here are the actual decisions, in order:

1. **Choose a family.** What property matters most — area, shape, or distance? That picks your compromise.

2. **Set the centre.** Where is your region of interest? Put the centre there. The sweet spot of the projection sits on your data.

3. **Set the parameters.** Standard parallels for conics, latitude of true scale for stereographic, datum for everything. These tune the projection to your specific region.

4. **Set the extent.** How much of the projected plane do you want to see? This is your map window.

<!-- [CODE BLOCK: build a CRS from scratch in R/Python, show each step:
     family → centre → params → extent → plot
     Use terra or sf for R, pyproj for Python
     Show the PROJ string growing: 
       +proj=laea 
       +proj=laea +lat_0=-42.88 +lon_0=147.33
       +proj=laea +lat_0=-42.88 +lon_0=147.33 +datum=WGS84
     Then crop to extent and plot] -->

That's four decisions. Not one. When someone says "use EPSG:3031" they've made all four decisions for you (family: stereographic, centre: south pole, true scale: 71°S, datum: WGS84) — which is fine if those choices match your problem, and quietly wrong if they don't. The point of understanding the components is that you can make each choice independently, for your data, for your question.

<!-- 
## Draft notes

- This opening replaces the "four things you need" framing from frag-ch02-crs-practical-problem.md,
  which becomes the SECOND section of Ch 2 (the "how to do it" part: three tool types, 
  reproject vs warp, the practical workflow)
- The projection-catalog-in-the-wild material from frag-ch02 slots in after this as 
  "what you'll encounter" — real datasets and why they chose what they chose
- Tissot indicatrix is the natural next topic: "we've said projections distort — 
  how do you measure that distortion?"
- The chicken-and-egg of extent/projection from frag-ch02 fits at the end
- Need figures badly. The developable-surface figure exists everywhere but the 
  "same family, different centres" figure is rare and would be genuinely useful
- The Mercator→Transverse Mercator→Oblique Mercator rotation sequence would be 
  a great animated figure or triptych
- Code should build the CRS incrementally, showing what changes at each step
-->
