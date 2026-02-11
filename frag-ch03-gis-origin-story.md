# How GIS divorced itself from topology

<!-- harvested from silicate proto-book (01-intro.Rmd, 02-limitations.Rmd) -->
<!-- target: Ch 3 "What does GDAL do?" or book introduction -->
<!-- status: DRAFT -->

## The compromise

When modern GIS was born in the 1990s it adopted a set of compromises that divorced it from its roots in graph theory. The earlier model — arc-node topology, where boundaries are shared edges in a network — was replaced by a model optimized for the most complicated cartographic and land-management data of the time. The path won. Geometry became sequences of coordinates grouped into rings and parts, stored whole, with no record of what they share with their neighbours.

The huge success of ArcView and the shapefile brought this arcane domain into common usage. The simple features standard, formalized in the early 2000s, codified the path-based view and removed some ambiguities from the shapefile spec. But it also locked in the limitations: shapes are paths, paths enclose space, and topology is something you compute on the fly and then throw away.

## What the standard excludes

Simple features has three structural limitations that matter for this book:

Shapes are represented as paths, so only planar polygonal surfaces are possible. You cannot represent a mesh, a TIN, or a non-planar surface. The standard allows XY, XYZ, and XYZM coordinate tuples, but this is not extensible — there is no capacity to store data against component geometry elements. A GPS track with temperature and salinity at each fix must store those measurements somewhere outside the geometry. And there is no capacity for internal topology: no vertex-sharing, no edge-sharing, no path-sharing between features. Every polygon stores its full boundary, even when it shares an edge with its neighbour.

These limitations mean simple features cannot represent in full the objects from GPS tracking, rgl meshes, ggplot2 aesthetics, spatstat point patterns, TopoJSON topology, CAD drawings, or 3D ecosystem models. Translations into and out of simple features are necessarily lossy for any of these sources.

## What GIS actually combines

Spatial, graphics, drawing-design, imagery, geodetic, and database techniques are each broader than any GIS, are used in combination in many fields, but no other field combines them in the way that GIS tools do. The GIS lens is visible whenever you try to cross the boundary:

Bringing an image into a GIS — localization metadata is assumed.  
Visualizing raster data with graphics tools — the georeferencing model doesn't transfer.  
Creating a line from a sensor platform that records temperature, salinity, position, and time — the line and its measurements live in different worlds.

The word "spatial" has a rather general meaning. GIS idioms sometimes extend into Z; time is usually special-cased. Where GIS really starts to show its limits is at the boundary between discrete and continuous: a raster cell that is simultaneously a value, an area, and a position. We prefer to default to the most general meaning of spatial, and reach for the GIS-specific optimizations only when they earn their constraints.

## The topology that was discarded

The irony is that the earlier GIS model *had* topology. Arc-node representations stored shared boundaries once, tracked adjacency explicitly, and allowed edits that propagated across features. That model was expensive to build and maintain, so the path-based alternative won on pragmatic grounds. But the topology didn't disappear from the problems — it disappeared from the data structures.

Every time sf computes `st_touches()` or `st_intersection()`, it internally reconstructs topological relationships, uses them, and discards them. The information exists transiently inside the algorithm but has nowhere to live in the output. You can detect that two polygons share an edge, but you cannot record *which* edge, attach attributes to it, or use it as a building block for further operations.

This is the gap that the relational decomposition approach fills. Not by replacing simple features — they remain the right choice for many workflows — but by providing a bridge to the indexed-primitive world where topology is explicit and geometry lives in tables.

<!-- 
## Source notes (not for publication)

The silicate book (mdsumner/sc-book, bookdown format, ~2017) was the first 
extended write-up of the PATH/PRIMITIVE decomposition idea. Package lineage:
spbabel (2015) → gris (2016) → rangl (2016) → silicate (2017–present).

The arc-node history reference: http://www.esri.com/news/arcuser/0401/topo.html

The specific claim about sf::st_touches() building and discarding topology 
is based on direct code reading — the GEOS predicates construct a topology 
internally via noding, use it for the predicate test, then return only the 
logical result.

The silicate book also contains extensive code demonstrating PATH and PRIMITIVE
models on sf objects, constrained Delaunay triangulation via RTriangle, and 
rgl mesh rendering. These are worked-example material for Ch 8, tracked in 
the manifest as reference.
-->
