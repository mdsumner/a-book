# The globe in the room

<!-- harvested from r-gris/rogue-book, 02-poly3d.Rmd -->
<!-- target: Ch 8 case study, or Ch 12 "Patterns for real work" -->
<!-- status: DRAFT -->

## From flat polygons to a 3D globe

The argument for decomposing geometry into tables can feel abstract until you try to do something that nested structures actively prevent. Putting polygons on a sphere is that something.

Start with a standard polygon dataset — country boundaries, the kind of thing every GIS tutorial begins with. In any nested-list representation (sp, sf), the geometry is opaque. You can ask for coordinates, but you get them as a side effect — extracted into a matrix, detached from everything else. You can't *modify* the coordinates in place without reconstructing the entire nested hierarchy.

Now try this: convert every vertex from longitude-latitude to Cartesian XYZ on a sphere. The maths is simple — a handful of trig functions:

```r
llh2xyz <- function(lonlatheight, rad = 6378137) {
  cosLat <- cos(lonlatheight[, 2] * pi / 180)
  sinLat <- sin(lonlatheight[, 2] * pi / 180)
  cosLon <- cos(lonlatheight[, 1] * pi / 180)
  sinLon <- sin(lonlatheight[, 1] * pi / 180)
  rad <- rad + lonlatheight[, 3]
  cbind(x = rad * cosLat * cosLon,
        y = rad * cosLat * sinLon,
        z = rad * sinLat)
}
```

With the geometry in tables, adding the third dimension is just adding columns. The vertex table has x and y; we compute X, Y, Z and store them right there, on the same rows, alongside the originals. No reconstruction, no recursion, no special methods. One `mutate()` call.

But plotting XYZ points is just a point cloud. To see polygons on a sphere, you need connectivity — which vertices are linked, which triangles tile which surfaces. This is where the table decomposition pays off a second time.

## From paths to meshes

A polygon boundary is a path: an ordered sequence of vertices forming a closed ring. To render that path as a filled surface in 3D, you need triangles. To get triangles, you need a Planar Straight Line Graph — the path decomposed into pairs of vertex indices, each pair defining one edge.

Building the PSLG from the table representation is mechanical: walk the branch table, pair consecutive vertex IDs, collect all segments. The vertex pool and the link table already have everything you need. With nested lists you'd be writing recursive extractors; with tables it's a grouped `lead()`/`lag()` operation.

Feed the PSLG to a constrained Delaunay triangulator (RTriangle in R, wrapping Shewchuk's Triangle library), and you get back a set of triangles defined as triples of vertex indices into your coordinate array — which is exactly the format that OpenGL (and rgl in R) wants.

The first result is ugly. The triangles span the full extent of each polygon with no internal vertices, so on a sphere they cut straight through the interior instead of following the curvature. They're flat facets stretched across what should be a curved surface.

The fix is to set a maximum triangle area in the triangulator. This forces it to add Steiner points — new vertices inside the polygon — ensuring no triangle is larger than a threshold. With enough internal structure, the triangles are small enough to follow the sphere's curvature. The polygons *drape*.

The moment that first globe rendered — country polygons in randomized greys, tessellated and wrapped around a sphere, rotating in an rgl window — was the moment the table decomposition stopped being a theoretical argument and became a visual one. You could see, literally, what the tables bought you: the ability to move between representations (path → PSLG → constrained triangulation → indexed mesh) without ever leaving the relational model, and the ability to add dimensions (lon-lat → XYZ → XYZ-on-terrain) by operating on the vertex table directly.

## The terrain extension

The same technique applies to real topography. Extract polygons for a region, project vertices to a local coordinate system, triangulate with a minimum area constraint, then look up elevation from a DEM at every vertex (including the Steiner points the triangulator added). Now your polygons aren't just on a sphere — they're draped over mountains and valleys, with the mesh density controlled by the area threshold.

This is what Manifold GIS did with its "terrain" objects, and what modern 3D tiling systems (Cesium 3D Tiles, Mapbox terrain) do at web scale. The principle is identical: geometry as indexed vertices, connectivity as integer references, and elevation (or any other attribute) as a column on the vertex table. The difference is that with the table decomposition, you don't need a specialized terrain engine — you need a triangulator and a vertex table.

<!--
## Implementation notes (not for publication)

Original code used: gris::gris() for table decomposition, gris::mkpslg() for PSLG 
construction, RTriangle::triangulate() for constrained Delaunay, rgl::triangles3d() 
for rendering.

Modern equivalent would use: silicate::PATH() or silicate::TRI(), or build PSLG 
directly from sf via sfheaders + manual segment pairing. anglr package wraps 
some of this. RTriangle still the go-to triangulator in R.

The "minimum area" parameter in RTriangle::triangulate(a = 9) is in the units 
of the input coordinates — here square degrees, which is wild but works for 
the demonstration.

Tasmania example used SRTM data + GADM boundaries + laea projection.
Never completed but the approach is sound.
-->
