# What Zarr cannot say

<!-- source: Michael, project thread 2026-10-01 -->
<!-- target: Ch 10 (virtual composition) or Ch 3 (formats), deferred chapter -->
<!-- status: SEED -->

Zarr presents itself as a representation for everything: n-dimensional arrays,
chunked, with coordinates. Two common cases stretch the rectilinear model past
where it holds.

UTM schemes. A collection of tiles in UTM is not one grid; it is sixty grids,
one per zone. To put it in one array you need zone as a dimension, and then the
x and y coordinates are only meaningful conditional on zone. The array model has
no way to say that a coordinate's meaning depends on another dimension's value.

Ragged time. Temporal coverage is naturally ragged: swaths, scenes and campaigns
do not land on a regular time axis. Forcing them onto one means padding with
missing values or inventing a nominal time, both of which lose the actual
sampling.

Both cases are relational, not rectilinear. A table of (zone, tile, chunk) rows
or (scene, time-range, chunk) rows says what happened without pretending the
world is a box. Tables with blobs (frag-ch08-arrays-are-tables-too) are the
honest representation; the rectilinear array is a view that works when it works.

Related, and in the same chapter: things that do not benefit from
virtualisation at all; the IFD structure of a COG as the oldest working example
of a chunk index; and storing chunked arrays in Parquet, which is tables with
blobs by another name.

<!--
Notes: this is the second-edition / deferred Ch 10 material. Keep it as a seed
until the first edition is out; the first-edition formats chapter should only
foreshadow it in a sentence.
-->
