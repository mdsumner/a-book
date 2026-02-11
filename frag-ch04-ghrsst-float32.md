# The GHRSST problem: when coordinates lie

<!-- harvested from mdsumner/rw-book ghrsst-netcdf.qmd -->
<!-- target: Ch 3 or Ch 4, as a case study -->
<!-- status: DRAFT -->

## A 648-million-pixel grid with noisy coordinates

GHRSST MUR is one of the most widely used oceanographic datasets: a daily, globally-complete sea surface temperature product at 0.01° resolution. The grid is 36000 × 17999 pixels. That dimension — 17999, not 18000 — is the first hint that something is off.

The file stores longitude and latitude as 1D coordinate arrays, as CF convention requires. But the coordinates are stored as Float32 (single-precision floating point), and when you look at the differences between consecutive values you find noise. The step *should* be exactly 0.01° everywhere. Instead it jitters in the sixth decimal place — not because the grid is irregular, but because 32-bit floats cannot represent the accumulating sum of 0.01 over 36000 steps without rounding errors.

Generate the same sequence in Float64 (double precision) and the steps are clean. Generate it in Float32 and you reproduce the noise in the file exactly. The coordinate arrays in GHRSST are not data — they're a lossy materialisation of what should be six numbers.

The metadata attributes claim the valid range is −180 to 180 in longitude and −90 to 90 in latitude. But 36000 × 0.01 = 360 and 17999 × 0.01 = 179.99. The grid doesn't quite reach from pole to pole. There's a missing row, and it's not clear whether it should be split half at the top and half at the bottom, or absorbed into the edge cells. The coordinate arrays are right-edge-aligned in longitude (ending at exactly 180°) and not quite symmetric in latitude.

These ambiguities — Float32 noise, a 17999-vs-18000 dimension, inconsistent alignment with the stated valid range — are invisible if you just call `xarray.open_dataset()` and plot. The data looks fine. The problems surface when you try to mosaic GHRSST tiles, align them with other datasets, or warp to a different CRS, and discover half-pixel offsets that shouldn't be there.

## The one-line fix

GDAL can cut through all of this with a single VRT connection string:

```
vrt://{source}?a_ullr=-180,89.995,180,-89.995&a_srs=EPSG:4326&a_offset=25&a_scale=0.001&sd_name=analysed_sst
```

This does four things at once: `a_ullr` assigns a clean, exact extent (ignoring the noisy coordinate arrays); `a_srs` declares the CRS; `a_offset` and `a_scale` apply the Int16-to-Celsius conversion so you get physical values directly; and `sd_name` selects the variable without the clunky `NETCDF:path:variable` prefix-suffix syntax (available since GDAL 3.9).

No intermediate files. No coordinate-array parsing. No Float32 precision hazards. The grid is declared as what it actually is — a regular 0.01° grid with a known extent — and GDAL's Warp API can then produce any subset, resolution, or projection you need.

This isn't a hack; it's the correct representation. The six numbers (36000, 17999, −180, 180, −89.995, 89.995) contain everything. The coordinate arrays in the file were always an inflation of this compact truth, and the Float32 storage was always going to corrupt them. The fix is to stop reading the corruption and start declaring the intent.

## The lesson

GHRSST is a cautionary tale about the cost of degenerate rectilinear storage. A regular grid was stored with materialised coordinate arrays in insufficient precision, creating phantom irregularity that no amount of downstream processing can fully repair. The fix isn't better interpolation or smarter coordinate parsing — it's recognising that the grid *is* regular and treating it as such.

Every global NetCDF product with Float32 lon/lat arrays on a 0.01° or finer grid has this problem to some degree. GHRSST is just the most visible because of its size and ubiquity. The general principle: if your grid is regular, store the geotransform (or extent + shape), not the coordinates. If your format requires coordinates, store them in Float64 or derive them at read time. And if you inherit a file where this wasn't done, assign the correct metadata externally rather than trusting the noisy arrays.

<!--
## Notes (not for publication)

The Float32 precision proof: in R, use cpp11 to generate coordinate sequences 
in both float and double, show that diff(float_coords) has noise matching the 
file while diff(double_coords) is clean. In Python, numpy.float32 arithmetic 
reproduces the same effect.

The 17999 dimension: most likely the producers started at -89.995 and stepped 
by 0.01 for 17999 steps, reaching 89.985. The "missing" row at 89.995 would 
make it 18000. Whether this was intentional (avoiding the pole) or a fencepost 
error is unclear from the metadata.

The a_scale=0.001 and a_offset=25 convert from the Int16 storage 
(add_offset=298.15, scale_factor=0.001 in the file, which gives Kelvin) 
to Celsius directly. The 298.15 K offset = 25°C offset at 0.001 scale.

GDAL's sd_name= syntax (since 3.9) is a huge ergonomic improvement for 
NetCDF/HDF workflows. Before that you needed NETCDF:"/path/to/file.nc":variable_name.
-->
