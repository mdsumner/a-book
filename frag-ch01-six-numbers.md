# Six numbers is all you need

<!-- harvested from mdsumner/rw-book georef.qmd and xarray_unref_matrix.ipynb -->
<!-- target: Ch 1 (What is a grid?) and Ch 4 (How formats encode "where") -->
<!-- status: DRAFT -->

## The raw matrix

Start with a matrix. 36 columns, 18 rows, values 0 to 647. In Python it's `np.arange(648).reshape([18, 36])`. In R it's `matrix(1:648, nrow = 18, byrow = TRUE)`.

Where is this matrix? Plot it and you'll get pixels — but positioned where? Some libraries default to unit square (0–1 in both axes), others use the array indices (0–36, 0–18). Neither is wrong. Neither is georeferenced. It's just a grid of numbers with no spatial meaning.

## Six numbers make it spatial

To turn this matrix into a geospatial dataset, we need six numbers: the shape (36, 18) and the extent (xmin = −180, xmax = 180, ymin = −90, ymax = 90). That's it. Each cell is 10° wide and 10° high. We have a global grid at coarse resolution.

These six numbers — ncol, nrow, xmin, xmax, ymin, ymax — contain *all* the georeferencing information for a regular grid. Everything else is a repackaging of the same content.

## The same information, six different ways

The geospatial world has invented at least six ways to express this:

**Extent**: xmin, xmax, ymin, ymax. The bounding box of the entire grid. Human-readable. Used by R's raster/terra packages, by GDAL's `-a_ullr` (in a different order: xmin, ymax, xmax, ymin — upper-left then lower-right), and by matplotlib's `imshow(extent=...)`.

**Offset and scale**: the position of one corner plus the cell size. Store the bottom-left corner (−180, −90) and the step (10, 10), and with the array shape you can reconstruct the extent. This is nice for stride-sampling (change the scale) or windowing (change the offset) without touching the data.

**Geotransform**: GDAL's six-element tuple. Top-left x, x-scale, x-shear, top-left y, y-shear, y-scale. For our grid: `(-180, 10, 0, 90, 0, -10)`. The two shear terms are zero for axis-aligned grids but handle rotation when they're not. This is the internal currency of GDAL — every format gets converted to and from this on the way in and out.

**Worldfile**: the ancient sidecar convention (.tfw, .jgw, .pgw). Six lines: x-scale, x-shear, y-shear, y-scale, x-centre-of-top-left-pixel, y-centre-of-top-left-pixel. Note the subtle difference: the worldfile references the *centre* of the corner pixel, not its edge. This half-pixel offset has caused more bugs than any other convention in geospatial computing.

**Rasterio Affine**: Sean Gillies chose `(x-scale, x-shear, x-origin, y-shear, y-scale, y-origin)` — a different parameter order from GDAL's geotransform, inherited from Casey Duncan's Planar library via the Affine class. Same information, transposed layout. The history of this ordering difference is a small but telling example of how Python's spatial ecosystem made design choices independently of GDAL's conventions.

**1D coordinate arrays**: materialise the cell centres (or edges) as explicit vectors. For our 36×18 grid, that's 36 longitude values and 18 latitude values instead of six numbers. This is what NetCDF/CF convention expects. It's what xarray stores internally. It's *degenerate rectilinear* — a regular grid inflated into a more general representation that adds no information but costs more storage and introduces the possibility of floating-point noise.

None of these is more correct than another. They all encode the same six degrees of freedom. But the choice has consequences: some representations survive format conversion cleanly, some accumulate precision errors, and some lock you into software assumptions that are hard to escape.

## Proving it works: hands on the metal

The most vivid way to understand georeferencing is to do it manually — no spatial libraries, no abstractions, just raw data and metadata.

**Write a file by hand.** Dump the matrix to a flat binary file (`.bil`), then write a six-line header (`.hdr`) declaring ncols, nrows, cellsize, xllcorner, yllcorner, and datatype. Open it with GDAL: it works. You have a georeferenced raster built from nothing but a numpy array and a text file. Strip the georeferencing from the header (set cellsize=1, xllcorner=0, yllcorner=0) and GDAL gives you back the identity geotransform. The format is just a container; the georeferencing is just metadata.

**Point GDAL at memory.** GDAL's MEM driver can open a numpy array directly by memory address, without writing any file at all. You construct a DSN string that declares the pointer, dimensions, datatype, and geotransform, and GDAL treats it as a dataset. This is terrifying and beautiful — it means the boundary between "an array in your program" and "a geospatial dataset" is literally just six numbers of metadata.

```python
def mem_dsn(x, extent):
    nrow, ncol = x.shape
    xmin, xmax, ymin, ymax = extent
    addr = x.__array_interface__["data"][0]
    gt = f"{xmin}/{(xmax-xmin)/ncol}/0/{ymax}/0/{(ymin-ymax)/nrow}"
    return (f'MEM:::DATAPOINTER="{addr}",PIXELS={ncol},LINES={nrow},'
            f'BANDS=1,DATATYPE=Float64,GEOTRANSFORM={gt}')
```

**Use GCPs.** Place two or three Ground Control Points — pixel positions paired with world coordinates — and GDAL can compute the geotransform by polynomial fit. For a regular grid this is overkill (two diagonal points suffice), but it demonstrates the general mechanism that also handles rubber-sheeting, orthorectification, and satellite sensor models. GCPs are resolved through the Warp API, not RasterIO — a distinction that matters when we get to the difference between "read pixels by index" and "resample to a new grid."

**Use a_ullr.** The simplest path: `gdal_translate input.bil output.vrt -a_ullr -180 90 180 -90`. Four numbers, upper-left then lower-right. Since GDAL 3.8, you can do this inline with the `vrt://` connection syntax: `vrt://input.bil?a_ullr=-180,90,180,-90`. No intermediate files, no API calls. This is GDAL at its most ergonomic.

Each of these methods arrives at the same place: a dataset with a geotransform. The format carried the data; the metadata positioned it. Understanding that these are independent concerns — and that the metadata can be assigned, reassigned, or derived from multiple sources — is the foundation of everything that follows.

<!--
## Notes (not for publication)

The MEM DATAPOINTER trick works in Python but is fragile in R because R's 
garbage collector can move objects. In R you'd use gdalraster::create() with 
the MEM driver instead.

The worldfile centre-vs-edge offset: GDAL reads worldfiles and internally 
adjusts to edge-based geotransform. This half-pixel correction is a perpetual 
source of off-by-half-pixel errors in legacy workflows.

The rasterio Affine ordering history: Casey Duncan's Planar library → 
Sean Gillies adopted for rasterio → now the standard in Python-spatial.
This is covered in more detail in the past conversation about Affine vs 
geotransform, which should become part of Ch 7.
-->
