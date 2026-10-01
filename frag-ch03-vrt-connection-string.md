# The logic moved into the string

<!-- rewritten 2026-10-01 from notes/gemini-vrt-transcript-2026-02.md (a Gemini session that produced three overlapping versions of this section) -->
<!-- target: Ch 3 (What does GDAL do?) -->
<!-- status: DRAFT -->

## One line, four handles

Here is a complete recipe for a 20 percent decimated crop of GEBCO global
bathymetry over Australia, read straight from the web:

```
vrt:///vsicurl/https://data.source.coop/alexgleith/gebco-2024/GEBCO_2024.tif?projwin=107,-10,150,-50&outsize=20%,20%
```

Three things are packed into that string. `/vsicurl/` tells GDAL to treat a
remote URL as a file and fetch only the byte ranges it needs. `projwin` crops at
the source, before any pixel is read. `outsize` decimates on the fly, using the
file's overviews if it has them. Nothing is downloaded until something reads
pixels, and then only the pixels asked for.

The string is the whole program. The language you hold it in is a detail:

```r
library(terra)
r <- rast(dsn)
```

```r
library(gdalraster)
ds <- new(GDALRaster, dsn)
m <- ds$read(band = 1, xoff = 0, yoff = 0,
             xsize = ds$getRasterXSize(), ysize = ds$getRasterYSize())
ds$close()
```

```python
import rasterio
with rasterio.open(dsn) as src:
    a = src.read(1)
```

```python
from osgeo import gdal
ds = gdal.Open(dsn)
a = ds.GetRasterBand(1).ReadAsArray()
ds = None
```

terra and rasterio are the comfortable handles; gdalraster and osgeo.gdal are the
bare ones, where you can see the dataset is a pointer and `read` is a `RasterIO`
call. All four open the same virtual dataset and get the same array, because the
crop and the decimation were done by GDAL's VRT driver, not by the package.

That is the point of this chapter. In a wrapper-heavy world you would have one
package to find the file, one function to crop, another to resample. Here the
logic is in the connection string, and the R and Python packages are different
ways of holding the handle of one tool.

## Crossing coordinate systems in the string

The same idea stretches across coordinate systems. Imagery from a tile server is
in Web Mercator; your study area is in longitude-latitude. Rather than reproject
anything by hand, declare which system your window is in:

```
vrt://WMTS:https://tiles.maps.eox.at/wmts/1.0.0/WMTSCapabilities.xml,layer=s2cloudless-2023?projwin=107,-10,150,-50&projwin_srs=EPSG:4326&outsize=20%,20%
```

`projwin_srs` says "the window I gave you is lon-lat; translate it to the source's
native coordinates before you fetch". Because this string and the GEBCO string use
the same `projwin` and `outsize`, the two arrays come back with the same
dimensions and stack directly. No reprojection code was written.

## From strings to pipelines

Connection strings are terse, and for anything more than a crop and a resize they
get hard to read. GDAL 3.10 and later add a second form: a streamed algorithm
serialised as JSON, which the GDALG driver opens as a live dataset.

```json
{
  "type": "gdal_streamed_alg",
  "command_line": "gdal raster convert /vsicurl/https://data.source.coop/alexgleith/gebco-2024/GEBCO_2024.tif --output-format stream -projwin 107 -10 150 -50 -outsize 20% 20%"
}
```

Save that as a `.gdalg.json` file, or build it as a string in memory, and pass it
to `rast()` or `rasterio.open()` like any other source. The `gdal raster pipeline`
command chains steps (`read ! warp ! calc ! write`) in the same way. The flags are
the ones you already know from the command line, the recipe is plain text that
crosses between R, Python, the shell and QGIS unchanged, and nothing touches disk
until you ask for output.

## The rule

If you find yourself writing code to manage tile downloads, crop windows or
resampling, stop and ask whether a connection string or a JSON pipeline already
does it. Usually one does. The intelligence is in the engine; the wrapper's job is
to hand over the string and receive the array.

<!--
## Notes (not for publication)

The original Gemini transcript (notes/gemini-vrt-transcript-2026-02.md) had three
versions of this plus a WMTS addendum. Kept: the four-handles matrix, projwin_srs,
GDALG JSON. Dropped: the chatbot's framing and follow-up questions.

Cross-references: frag-ch04-ghrsst-float32 uses the same vrt:// mechanism with
a_ullr / a_srs / a_offset / a_scale / sd_name. frag-ch01-six-numbers introduces
vrt://?a_ullr. Worth a cheat-sheet appendix of vrt:// parameters: projwin,
projwin_srs, outsize, ovr, a_ullr, a_srs, sd_name, bands, expand, if.

The WMTS example needs checking against the current EOX capabilities URL and
layer name before it ships.
-->
