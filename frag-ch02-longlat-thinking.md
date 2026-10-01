# Longlat thinking

<!-- source: Michael, project thread 2026-10-01 -->
<!-- target: Ch 2 (What is a CRS?) closing section, or Ch 4 grid taxonomy -->
<!-- status: SEED -->

Everyone knows Web Mercator is the wrong map. Fewer people notice that the
NetCDF and array world defaults to the same mistake in a different costume:
longlat thinking. A global 0.25 degree grid is treated as if it were the Earth,
coordinates are degrees, and area, distance and neighbourhood are read straight
off the array indices.

A regular grid is a regular grid. Its cells are equal in index space, and that is
all the array knows. The relationship between that grid and the oblate spheroid
is a separate topic, and it is a coordinate transformation, not a property of the
data. Longlat is a projection with a family (equirectangular), a centre (0, 0)
and a datum, and it distorts area as badly as Mercator does near the poles, just
in a different pattern. Mercator-webmap thinking and longlat-array thinking are
the same error: mistaking one particular projected grid for the sphere.

This is why the OISST example (frag-ch02-equal-area-warp-stats) needs cos(lat)
weights in longlat and none in LAEA. The weights are not a correction for a
defect in the data. They are the Jacobian of a projection that was chosen by
default and never named.

<!--
Notes: ties to grid taxonomy case (C) degenerate rectilinear, and to the Ch 2
family/centre decomposition: longlat is +proj=longlat with lon_0=0, and it has
a centre like everything else. Also to the Zarr seed (frag-ch10): the array
model's default to longlat is a special case of assuming the world is
rectilinear.
-->
