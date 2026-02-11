# Three kinds of triangulation

<!-- harvested from r-gris/rogue-book 02-poly3d and 09-triangulation-in-R -->
<!-- target: Ch 8 (geometry as topology) or Ch 9 (sparse structures), as a sidebar/box -->
<!-- status: SNIPPET -->

Once you have a vertex pool and a set of edges (a PSLG), you can triangulate — but the choice of algorithm determines what you get.

**Delaunay triangulation** satisfies the "empty circumcircle" condition: no vertex falls inside the circumscribed circle of any triangle. This maximizes the minimum angle across all triangles, producing well-shaped elements. It's the right choice for interpolation and surface fitting, because well-shaped triangles give stable numerical results. But it ignores your input edges — polygon boundaries may be crossed by Delaunay edges, which means the triangulation doesn't respect the shapes you started with.

**Constrained Delaunay triangulation** keeps all your input edges intact while being "as Delaunay as possible" everywhere else. You can also impose constraints on minimum triangle area and minimum interior angle, forcing the triangulator to add Steiner points (new vertices) to meet quality targets. This is what you want for meshing polygons: boundaries are preserved, interior density is controllable, and the resulting triangles are well-shaped enough for rendering and computation. In R, RTriangle wraps Shewchuk's Triangle library for this. In Python, triangle and meshpy provide similar access.

**Ear-clipping** is the fast, simple alternative. Walk the polygon boundary, find "ears" (triangles formed by three consecutive vertices where no other vertex falls inside), clip them off, repeat. It's O(n²), always preserves input edges, handles non-convex polygons, and needs no external library — rgl includes one. But it produces no interior vertices and makes no quality guarantees. The resulting triangles can be arbitrarily thin or oddly shaped. It's fine for rendering flat polygons (where triangle quality doesn't matter for visual appearance) but poor for surface fitting or finite-element work.

The choice cascades from the application: ear-clipping for quick visualization, constrained Delaunay for meshing and draping, unconstrained Delaunay for interpolation from scattered points.

<!--
R packages: RTriangle (constrained Delaunay), rgl (ear-clipping built in), 
deldir (Delaunay + Voronoi), interp (Delaunay), geometry (n-dimensional via Qhull).
Python: scipy.spatial.Delaunay (unconstrained), triangle (constrained), 
meshpy (constrained, Shewchuk), mapbox_earcut (ear-clipping).

The decido package provides ear-clipping for R via the mapbox earcut algorithm,
faster than rgl's built-in for large polygons.
-->
