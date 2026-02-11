# Rosetta Stone: entity terminology across spatial systems

<!-- harvested from r-gris/rogue-book 01-intro, updated for 2026 -->
<!-- target: Ch 8, as a reference table early in the chapter -->
<!-- status: SNIPPET — may merge into frag-ch08-geometry-is-tables.md -->

The same structural concepts recur across every spatial system, wearing different names. This table maps them:

| Concept | PostGIS | sp | sf | ggplot2 fortify | terra::geom | GDAL | silicate | General term |
|---------|---------|-----|-----|-----------------|-------------|------|----------|-------------|
| A thing with attributes | feature | Spatial*DataFrame row | sf row | `id` | `object` | Feature | object | **Object** |
| A connected piece of a thing | element | Polygons/Lines slot | ring/linestring within geometry | `piece`/`group` | `part` | Ring/subgeometry | path | **Branch** |
| A location | point | coordinate matrix row | coordinate | `long`/`lat` | `x`/`y` | point | vertex (unique) or coord (instance) | **Vertex** |
| A topological atom | — | — | — | — | — | — | segment (1D) / triangle (2D) | **Primitive** |

Notes:

- sp conflated branches and objects at the class level (Polygons vs Polygon, Lines vs Line) but didn't give branches first-class identity — you couldn't attach attributes to a ring.
- sf improved the hole model (first ring = exterior, subsequent = holes) but kept geometry opaque via the `sfc` list-column. Branches still aren't queryable entities.
- The **Primitive** row is empty for most systems because they don't decompose below the branch level. This is precisely the gap that silicate, anglr, and the vertex-pool approach fill.
- GDAL's internal model aligns most closely with the PostGIS column: features contain geometries, geometries contain rings/subgeometries, rings contain points.

<!-- 
Consider: add gdalraster, wk, and geos columns? They map slightly differently:
- wk: wk_handle() streams vertices, no named entity levels
- geos: GEOSGeometry, GEOSCoordSequence — two levels, no branch concept
- gdalraster: inherits GDAL model via OGR
-->
