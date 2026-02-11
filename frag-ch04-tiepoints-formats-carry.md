# Formats carry, libraries warp

<!-- harvested from mdsumner/gdal-r-python georef-tiepoint/raster-coordinate.md + tiepoint-illustrate-expanded.md -->
<!-- target: Ch 4 (How formats encode "where") — the centrepiece -->
<!-- status: DRAFT -->
<!-- note: Ryan Abernathey conversations shaped the two-worlds framing -->

## The tiepoint that became a geotransform

GeoTIFF emerged in 1994–95 from a practical problem: TIFF files had no way to say where their pixels were on Earth. The solution was a tiepoint — six numbers `(I, J, K, X, Y, Z)` pairing a pixel position with a world coordinate — plus a pixel scale tag declaring cell size. One tiepoint plus scale is mathematically equivalent to a six-parameter affine geotransform. Same information, different encoding.

The spec also defined a full 4×4 transformation matrix (ModelTransformationTag) for rotation and shear, and mentioned the possibility of multiple tiepoints for non-affine distortions. But that multiple-tiepoint case was explicitly marked "not Baseline GeoTIFF" and "should not be used for interchange." The spec went further: it refused to define any interpolation method for points between tiepoints, noting that "defining such a set of bounding tiepoints does not imply that the model space locations of the interior of the image may be exactly computed by a linear interpolation of these tiepoints."

This was a deliberate design choice, and it was the right one. The format stores knowledge — these pixels correspond to these coordinates — but the application applies judgment about how to interpolate between them.

Empirical evidence confirms the wisdom of this restraint. A survey of GDAL's autotest suite finds zero test cases for multiple tiepoints in ModelTiepointTag. The rubber-sheeting use case was solved outside the format entirely: ArcGIS stores GCPs in `.aux.xml` sidecars, GDAL stores them in its own metadata domain. The GeoTIFF tag designed for non-affine georeferencing was never used for it in practice.

## Four lineages of "where is this pixel?"

The geospatial world developed at least four distinct mechanisms for tying rasters to coordinates, each with different mathematical properties:

**Affine geotransform.** Six parameters: offset, scale, optional shear. Fully determined — given the parameters, every pixel has an exact world coordinate. This is the baseline case, the "goal state" for rectified products, and what GDAL uses internally. Any reader can implement it.

**Ground Control Points.** Sparse point pairs mapping pixel positions to world coordinates, requiring an interpolation method (polynomial order 1–3, thin plate spline, homography) to fill the gaps. The same set of GCPs produces different results depending on the method chosen. GCPs emerged as a post-hoc correction mechanism — you have an unrectified image and you're telling the software where known points should end up.

**Rational Polynomial Coefficients (RPCs).** A "black box" provided by satellite vendors that maps (latitude, longitude, height) to (row, column) via rational polynomials. RPCs are fundamentally 3D: you cannot evaluate them without a DEM providing ground elevation at each point. This makes them categorically different from affine or GCP models — they are not self-contained georeferencing.

**Geolocation arrays.** Explicit coordinate values for every pixel (or a dense subsampled grid) stored as 2D longitude and latitude arrays. No interpolation ambiguity — the coordinates are the data. This is how satellite swath sensors (AVHRR, MODIS, Sentinel-3) and ocean/atmosphere model grids naturally describe their geometry.

These four approaches are not interchangeable. They have different mathematical properties, different data volumes, different use cases, and different assumptions about what "georeferencing" means.

## Two worlds, two mental models

The deepest tension in geospatial format design isn't technical — it's philosophical. Remote sensing and climate/ocean modelling have fundamentally different relationships with coordinate systems.

**The remote sensing mental model:** raw imagery needs geometric correction. The sensor captured pixels; you warp them to a map projection. The geotransform describes the corrected state. Unrectified data is an intermediate. "Rectification" is the essential processing step. The affine geotransform is the goal; GCPs and RPCs are steps along the way.

**The climate/modelling mental model:** model output lives on a computational grid. That grid IS the coordinate system. When a climate modeler has NEMO output on a rotated-pole ORCA grid with 2D `nav_lat`/`nav_lon` arrays, they're not thinking "this needs rectification." The curvilinear grid is the authoritative representation. Regridding to regular lon/lat is an analytical choice — necessary for some comparisons, but fundamentally a transformation that loses information about the native grid structure. Nobody in the CF world talks about "rectification."

This explains the architectural difference between the ecosystems. In the remote sensing world, GDAL's Warp API is the essential tool — it applies the correction that transforms raw imagery into map products. In the climate world, xarray regridding (via xESMF, pyresample, CDO) is an explicit analytical operation with user-specified methods. There's no "auto-warp on read" in the NetCDF ecosystem because the data is already where it's supposed to be.

CF conventions treat 2D auxiliary coordinate variables as a complete georeferencing solution, not a deficient form waiting to be converted to a geotransform. The `lat(y,x)` and `lon(y,x)` arrays aren't a workaround — they're the complete description of where each grid cell is located. The curvilinear structure is intentional.

## GDAL as Rosetta Stone

GDAL's unified transformer API (`GDALCreateGenImgProjTransformer2`) presents a single interface across all four mechanisms: affine geotransform, GCP polynomial/TPS, RPCs (with DEM), and geolocation arrays. This is the practical unification that no format spec has achieved.

But it comes with implicit choices. GDAL's default precedence — geotransform first, then GCPs, then geolocation arrays — means that data with multiple georeferencing mechanisms gets handled by silent priority rules. And the fundamental distinction between RasterIO (read pixels, carry metadata through) and Warp (apply transformation, resample to new grid) maps directly to the format-vs-library boundary: formats store the "what and where," the warper implements the "how to transform."

## What this means for GeoZarr

GeoZarr sits at the intersection of both worlds. It inherits CF conventions from the climate side and aims to support satellite imagery products from the remote sensing side. The risk is trying to encode all four georeferencing models within the format specification itself — defining interpolation methods for GCPs, specifying what readers should do with RPCs, standardising warping semantics.

The GeoTIFF spec got this right by explicitly putting interpolation "out of scope." A format should be a description of state, not a transformation of state. The format carries; the library warps.

GeoZarr should be a "Container of Intent" — a standardised way to carry each georeferencing lineage's native metadata cleanly, without trying to merge them into a single unified model or specify transformation semantics. Support the affine model for rectified products. Support coordinate arrays for climate/model grids. Carry RPC metadata for satellite products. But don't try to define what readers do with non-affine georeferencing, because that's the library's job and different libraries will (correctly) make different choices.

## The spectrum in one table

| Model | Parameters | Determinism | Requires | Typical source |
|-------|-----------|-------------|----------|---------------|
| Affine | 6 numbers | Fully determined | Nothing | Rectified imagery, map products |
| GCPs | Sparse point pairs | Interpolation-dependent | Method choice | Manual registration, historical imagery |
| RPCs | Rational polynomials | Determined given elevation | DEM | Satellite vendors (DigitalGlobe, Pléiades) |
| Geolocation arrays | Explicit per-pixel coords | Fully determined | Nothing | Swath sensors, model output |

The progression from affine through geolocation arrays is a progression from compact-but-constrained to verbose-but-general. The affine model encodes a regular grid in six numbers. Geolocation arrays encode an arbitrary grid in millions of coordinate pairs. Between them, GCPs and RPCs occupy an awkward middle ground — more flexible than affine, but requiring external decisions (interpolation method, elevation data) that the format cannot provide.

Understanding which model your data actually uses — and which model your software thinks it uses — is the first step to avoiding the class of bugs that haunt operational geospatial workflows: off-by-half-pixel errors, wrong projections, silent resampling with unintended methods.

<!--
## Notes (not for publication)

The tiepoint-illustrate-expanded.md companion has full Python and R code 
for generating comparison plots of all six georeferencing variants (A through F). 
Worth including as a figure or interactive element in the book.

The PixelIsPoint vs PixelIsArea distinction (the half-pixel demon) connects 
directly to frag-ch01-centre-vs-edge.md. Cross-reference.

The GeoZarr "Container of Intent" framing is Michael's hypothesis, flagged 
as needing verification against current spec drafts and SWG discussions. 
The two-worlds analysis was developed in conversation with Ryan Abernathey 
and is complementary to the grid taxonomy in frag-ch04-grid-taxonomy.md.

The cell semantics section (CF bounds convention, n+1 corner arrays) was 
cut from this fragment but exists in full in raster-coordinate.md Section 6.4. 
Worth revisiting when drafting Ch 4 in full.

GDAL RFC 4 (geolocation arrays) and RFC 22 (RPCs) are the key primary sources.
The empirical evidence from GDAL's autotest suite (February 2026) is original 
research — worth preserving the methodology.
-->
