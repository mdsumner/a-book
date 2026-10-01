# The array objection

<!-- source: Michael, project thread 2026-10-01 -->
<!-- target: Ch 8 (geometry as topology), after "What the tables buy you" -->
<!-- status: SEED -->

The shade cast on "geometry is tables" does not only come from the GIS side. The
array community casts it too, and the strawman is the same each time: a fully
materialised table of every value and every coordinate is not practical, so tables
are for small data and arrays are for real data.

That argument ignores two things. First, a table cell can hold an encoded chunk.
A row that says "tile (3, 7), compressed bytes, these dimensions, this dtype" is a
perfectly good row, and a table of such rows is what every chunked array store
already is underneath, whether it admits it or not. Second, the thing that makes
tables work is not materialisation, it is relations. Arrays have the same secret:
dimensions are shared keys, coordinates are lookup tables, a labelled array is a
join waiting to happen. Nothing has to be one to one. A vertex table and a chunk
table are the same kind of object, differing only in what the key points at.

So the table decomposition is not an alternative to arrays. It is the description
of how arrays and geometries relate to each other, and it is the part both camps
leave implicit.

<!--
Notes: ties to the grid taxonomy (ch04) where xarray materialises coordinates and
GDAL keeps six numbers: that is the same materialise-vs-relate choice. Also ties
to ch09 sparse structures: the rasterized-polygon intermediate is a table of
(cell, weight) rows, i.e. a relation between a geometry and an array.
-->
