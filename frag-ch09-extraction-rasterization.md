# Extraction and rasterization are the same operation

<!-- harvested from mdsumner/rw-book experiments/complement.Rmd -->
<!-- target: Ch 9 (Grids as sparse structures), to be merged with gridburn fragments later -->
<!-- status: SNIPPET — standalone until gridburn docs-design is re-harvested -->

When exactextract runs on a polygon layer, it does something specific: for each polygon, it detects which raster cells intersect the boundary (using GEOS polygon-pixel overlay), flood-fills the interior to find fully-contained cells, then aggregates the raster values weighted by the fractional overlap. The boundary cells get weights between 0 and 1; the interior cells get weight 1.

That intermediate state — a set of boundary cells with fractional weights and interior cells with weight 1 — *is* a polygon rasterization. It's a sparse representation of the polygon on the grid: boundary cells explicitly stored, interior cells encodable as run-length spans per row. If you keep this intermediate instead of discarding it after aggregation, you have a compact rasterized polygon that supports both exact extraction (multiply by weights, sum) and approximate methods (cell-centre-in-polygon, which is what fasterize does at much higher speed).

Extraction and rasterization are complements. Rasterization is the process of finding which cells a polygon covers; extraction is the process of reading values from those cells. Every extraction implicitly rasterizes; every rasterization implicitly defines an extraction mask. The sparse storage of the boundary-and-fill result is the bridge between them — and it's the same structure that underpins conservative regridding, where the weight matrix describes how source cells map to target cells.

The consolidation — approximate method for speed, exact method when needed, rasterization and extraction unified through a shared sparse representation — is what gridburn aims to provide. The same principle applies to lines: rasterize a line to the cells it crosses, and you have both a line-rasterization and a transect-extraction mask.

<!--
## Notes (not for publication)

Dan Baston (exactextract author) confirmed the architecture but noted 
the abstraction is "too abstract for the main users of exactextract."
The intermediate storage idea is what controlledburn (hypertidy) explored 
with approximate methods and what gridburn now pursues with exact methods 
by vendoring exactextract's core algorithm.

Conservative regridding (ESMF, xESMF) uses the same weight-matrix concept 
but in a grid-to-grid context rather than polygon-to-grid. The connection 
is worth making explicit in the book.
-->
