# CRS is a practical problem, not a theoretical one

<!-- harvested from antequated.org (rbind/antequated.org): polar-maps-now.Rmd (2017), map-projections.qmd (2023), get-antequated-with-somap.Rmd (2019) -->
<!-- target: Ch 2 "What is a CRS?" -->
<!-- status: DRAFT -->

## Four things you need

To make a projected map you need exactly four things: a data manipulation environment, coordinate transformation tools, a coordinate reference system specification, and data. Every complexity in this chapter is a variation on one of these four.

The critical first step is knowing what coordinate system your data is already in. A dataset in longitude-latitude is not "unprojected" — it uses a specific coordinate system with specific assumptions about the ellipsoid and datum. The word "unprojected" is misleading because it implies the data has no spatial reference. It does. If your software doesn't know what that reference is, the transformation will fail or — worse — succeed silently with wrong results.

## Three types of projection tools

There are three fundamentally different operations, and confusing them is a common source of bugs:

**Reproject points** from source CRS to target CRS. This is pure coordinate math — take (lon, lat) pairs, apply a transformation, get (x, y) pairs. Fast, exact (within the limits of the datum transformation), and the simplest operation.

**Reproject vector objects.** The object stores its source CRS as metadata. The operation transforms all coordinates and updates the metadata. If anything goes wrong, the process should fail completely — a half-transformed object with wrong metadata is worse than an error. This is what `spTransform()` did and what `sf::st_transform()` does.

**Warp rasters.** This is fundamentally different. You can't just transform the corner coordinates of a raster, because the grid cells in the source CRS don't map to grid cells in the target CRS. Warping creates a new regular grid in the target CRS and resamples values from the source — it's interpolation, not just coordinate math. GDAL's warper handles this, and it's now so fast and accessible that it's used routinely for standardizing data streams and interactive visualization.

The practical rule: reproject your points and vectors *to the raster's CRS* for extraction, not the other way around. Warping is expensive and introduces resampling artifacts. But when you need a common grid across multiple sources, warping is the right tool.

There's a wider remit of *regridding* with complex coordinate arrays (climate models, ocean models), but that's a different conversation — some of those grids aren't really curvilinear, and some use storage modes (tabular XYZ) that are problematic for both size and fidelity.

## Projection families and parameters

A projection specification like `+proj=stere +lat_0=-90 +lon_0=142 +datum=WGS84` has two kinds of information. The *family* (`+proj=stere`, Polar Stereographic) is the mathematical model — a particular set of compromises between area, distance, and angle. The *parameters* (central longitude, central latitude, datum, ellipsoid, latitude of true scale) orient that model relative to the Earth.

There is a literal infinity of possible Polar Stereographic projections. In practice we use a handful, but the distinction matters: changing `lon_0` from 142 to 10 is a rotation of the same mathematical surface; changing `+proj=stere` to `+proj=laea` is a completely different set of trade-offs.

The three fundamental compromises: **equal-area** (area is correct everywhere in the projection — good for density maps, sea ice extent), **equidistant** (distance is correct along one axis or from one point — less commonly needed), **conformal** (angles and local shapes are preserved — good for navigation, flow fields). No projection achieves all three.

The other unsaid thing here is that a projection or a crs is not a map, for a map we also need a bounding box and that is not a trivial task (see chicken egg below) but also this can lead into some code examples that make it clear that bbox is a bit rubbery 'Tasmania' (mainlaind or all offshore islands), and cropping too early is its own kind of problem for data manipulation. 

## Projections in the wild

Many datasets arrive already projected, and knowing what projection they use — and why — is part of understanding the data:

**Polar Stereographic** — NSIDC sea ice products, SSMI brightness temperatures. Conformal, so ice edge shapes look right. Different hemispheres use different parameters: south pole at `+lat_0=-90 +lon_0=0`, north pole with Hughes 1980 ellipsoid at `+lat_0=90 +lon_0=-45`.

**Lambert Azimuthal Equal Area** — CCAMLR management zones, many AAD Antarctic datasets. Equal-area, so statistical summaries over regions give correct results.

**Mercator variants** — AVISO ocean currents, Smith/Sandwell bathymetry, web tile services. The old AVISO NetCDF stored currents on a Mercator grid but presented them as if they were longitude-latitude — the grid looked right when plotted naively, but the map overlay didn't line up until you recognized it was rectilinear in lon-lat (latitude lines denser toward poles) but regular in native Mercator projection.

**Equal Area Cylindrical** — Fraser/Massom fast ice maps around Antarctica, with a custom central longitude and latitude of true scale.

**Sinusoidal** — SeaWiFS and MODIS Level 3 ocean colour. An equal-area choice that tiles the globe differently from cylindrical projections.

Some complex models (like WAOM2 ROMS for the Southern Ocean) are a regular grid in a projection but are delivered with longitude-latitude coordinate arrays. The grid *is* regular — in the model's native projection. The lon-lat arrays are a convenience that hides this fact and makes the data look curvilinear when it isn't. Other models are truly curvilinear (no underlying regular grid), and the distinction matters for how you extract values from them.

## The chicken-and-egg of extent and projection

Making a map involves circular dependencies. You choose a projection, which determines how space is distorted, which affects what extent looks reasonable, which affects the projection choice. You pick an extent, but then want more or less, which changes the projection parameters, which subtly impacts the graticule, labels, and data overlay.

This back-and-forth is inherent, not a sign that your tools are broken. The more familiar you are with the component tools, the faster you iterate. SOmap automated some of this for Antarctic maps — `SOauto_map()` adapts both projection and extent to the input data — but automation only gets you a starting point. The finishing is always manual.  

On scale bars and north arrows: a scale bar is only accurate if you orient it where linear distance on the ground corresponds to equal spacing in the map, which is rare. A north arrow is either totally redundant (north is up, the graticule shows it) or hopelessly misleading (in a polar projection, "north" points in every direction from the pole). Exploring what's wrong with these cartographic conventions leads directly to understanding the Tissot indicatrix — the formal measure of how a projection distorts the local geometry at every point on the map.

<!-- 
## Source notes (not for publication)

antequated.org was a blogdown site on Yihui Xie's rbind GitHub org, 
dedicated to polar mapping techniques. Three original posts by Michael 
(2017, 2019, 2023) plus one by Jaimie Cleeland (Southern Ocean map tutorial).

The 2023 map-projections post is the most mature — it has the three-tool-types 
taxonomy, the real-world projection catalog, and the Mercator grid gotcha.
The 2017 post has the sharper pedagogical framing (four ingredients, 
"unprojected" misnomer). The 2019 SOmap post demonstrates auto-projection.

The Southern Ocean map tutorial (Cleeland, 2017) is a worked example using 
sp/rgdal/raster era tools — good reference for a Ch 2 or Ch 12 worked example 
but conceptual content is covered here.

Tissot indicatrix package: https://github.com/mdsumner/tissot (now being 
prepared for CRAN as the tissot package with tissot_raster() for computing 
distortion metrics on regular grids in projected coordinate systems).

The projection catalog (NSIDC stereo, CCAMLR LAEA, AVISO Mercator, etc.) 
comes from direct experience at AAD working with these datasets via raadtools.
-->
