# The half-pixel problem

<!-- harvested from mdsumner/rw-book r-image.qmd -->
<!-- target: Ch 1 (What is a grid?) as a sidebar or callout -->
<!-- status: SNIPPET -->

R has had this confusion baked into base graphics since at least 1998. The `image()` function takes 1D vectors of cell *edges* (n+1 values for n cells) to position a matrix. The `contour()` function takes 1D vectors of cell *centres* (n values for n cells). They work on the same data but expect different coordinate conventions, so plotting a contour over an image requires converting from edges to centres — a half-pixel offset that's easy to get wrong.

Meanwhile, `rasterImage()` takes an extent (xmin, ymin, xmax, ymax) — the affine model, no coordinate arrays at all. Three base R functions, three conventions for the same grid.

This isn't an R quirk. It's the same centre-vs-edge ambiguity that appears in the worldfile format (centres) vs the geotransform (edges), in NetCDF coordinate arrays (centres, usually) vs GDAL's extent model (edges), and in the Alvy Ray Smith memo "A Pixel Is Not A Little Square" that has been trying to sort this out since 1995. The question "does this coordinate refer to the centre of the cell or the corner?" has no universal answer, only local conventions — and every conversion between conventions is an opportunity for a half-pixel error.

<!--
volcano dataset: 61x87 matrix, 10m cells, extent 2667400,2668010,6478700,6479570 
on NZGD49 (EPSG:27200). The fact that even this toy dataset requires careful 
handling of centre-vs-edge to align correctly with imagery is telling.
-->
