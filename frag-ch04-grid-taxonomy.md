# A taxonomy of grid georeferencing

<!-- harvested from mdsumner/rw-book grids.qmd, refined in conversation with Ryan Abernathey -->
<!-- target: Ch 3 (What does GDAL do?) or Ch 4 (How formats encode "where") -->
<!-- status: DRAFT -->

## Eight modes of the same grid

A single regular grid can appear in at least eight different forms depending on how its georeferencing is stored. Some of these are genuinely different data structures; others are the same information in a more expensive encoding. Telling them apart matters, because software makes different assumptions about each.

**(A) Non-georeferenced array.** A matrix with no spatial metadata. Positioned at the origin with unit cell size by convention. This is what you get when you read a PNG or a raw binary without a sidecar.

**(B) Affine-georeferenced array.** The compact form: a geotransform (offset, scale, optional shear) positions the grid in world coordinates. This is GDAL's native representation and the most information-efficient. Six numbers for any axis-aligned grid.

**(C) Degenerate rectilinear.** The regular grid from (B), but with cell-centre coordinates materialised as 1D arrays. Thirty-six longitude values and eighteen latitude values instead of six numbers. This is what NetCDF/CF convention requires and what xarray stores internally. It is *degenerate* because the arrays are a perfectly regular sequence that could be reconstructed from extent and shape alone. The extra storage adds no information but opens the door to floating-point noise.

**(D) Degenerate rectilinear from a projection.** A grid that *is* regular in its native projected coordinate system, but has been stored as 1D longitude and latitude arrays after transforming the coordinates. The classic example: AVISO ocean current data on a global Mercator grid, stored in NetCDF with non-uniform latitude steps. The latitude array looks rectilinear (genuinely varying step size) but the underlying grid is perfectly regular in Mercator metres. If you know the projection, you can recover the compact form. If you don't, you're stuck treating it as truly rectilinear.

**(E) Degenerate curvilinear from a projection.** Every cell gets its own (lon, lat) pair stored in 2D arrays, even though the grid is regular in projected space. This happens with satellite swath data reprojected to a regular grid but stored with explicit lon/lat geolocation arrays. The 2D arrays scream "curvilinear!" but the underlying geometry is a regular grid wearing a coordinate-transform costume.

**(F) Degenerate curvilinear, native coordinates.** The inverse pathology: 2D arrays of projected (x, y) for a grid that's regular in those same projected coordinates. Rare in practice, but theoretically possible and would be deeply confusing.

**(G) Truly rectilinear.** A grid with genuinely non-uniform spacing in at least one dimension. The step sizes vary and cannot be compacted to offset + scale. The 1D coordinate arrays carry real information. Many ocean and atmosphere model grids fall here, with spacing that increases toward poles or boundaries.

**(H) Truly curvilinear.** Every cell has a unique (x, y) position that follows no regular pattern in any coordinate system. Rotated-pole grids, tripolar ocean grids, satellite swath geometries. The full 2D coordinate arrays are necessary and irreducible.

## How software sees this world

The central tension: xarray and GDAL look at these eight cases and make opposite default assumptions.

**xarray assumes rectilinear or curvilinear.** Every dataset gets 1D or 2D coordinate arrays. If the source is a regular grid (B), xarray materialises it to (C) on load — there is no internal representation for "this is regular, trust the six numbers." This works beautifully for (G) and (H), where the coordinates carry real information. For (B), (C), and (D), it's a lossy expansion that discards the compact representation and introduces precision hazards.

**GDAL assumes affine.** If a dataset is georeferenced, GDAL converts it to form (B) internally. If it finds 1D or 2D coordinate arrays, it offers two pathways: RasterIO (ignore the coordinates, use the geotransform for pixel indexing) or Warp (use the coordinates as geolocation arrays, resample to a target grid). For cases (D) and (E), this means GDAL can recover the true regular grid through warping — but only if you tell it the target CRS and resolution.

Neither worldview is wrong. xarray is optimised for labelled-array operations where coordinates are first-class dimensions. GDAL is optimised for raster I/O and reprojection where the geotransform is the source of truth. The problems arise at the boundary: when xarray opens a GDAL-referenced dataset through rioxarray, it materialises the geotransform to coordinate arrays, losing compactness. When GDAL reads a NetCDF that xarray wrote, it may find coordinate arrays that are regular-but-noisy and has to decide whether to trust them or derive a clean geotransform.

Understanding which of the eight modes your data is actually in — and which mode your software *thinks* it's in — is the first step to avoiding the off-by-half-pixel, wrong-projection, noisy-coordinate class of bugs that haunt operational geospatial workflows.

<!--
## Notes (not for publication)

The taxonomy emerged from conversations with Ryan Abernathey about how 
xarray and GDAL see the same data differently. Ryan's perspective from the 
Pangeo/xarray side was that labelled coordinates are the right default; 
Michael's from the GDAL side was that the geotransform is more compact and 
less error-prone. Both are right for their contexts.

The AVISO Mercator case (D) is a perfect pedagogical example: 
the file looks rectilinear, IS regular in Mercator, and xarray has no 
way to know this without external information. GDAL can resolve it if 
you tell it to warp from the geolocation arrays to a Mercator target grid.

Cases (E) and (F) are less common but important for understanding satellite 
data products. Many L2 swath products are (H) genuinely, but some L3/L4 
products are (E) — regular in projected space but stored with longlat 
geolocation arrays "for convenience."
-->
