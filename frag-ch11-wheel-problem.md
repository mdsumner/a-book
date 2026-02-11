# The wheel problem

<!-- harvested from mdsumner/gdal-r-python installation/gdal-python-binaries.md -->
<!-- target: Ch 11 (The installation problem) -->
<!-- status: DRAFT -->

## Why `pip install GDAL` doesn't work

The `GDAL` package on PyPI ships only source distributions. Installing it requires a pre-installed GDAL C library with matching development headers, plus a compiler toolchain. This makes it one of the most common pain points in the Python geospatial ecosystem — and the reasons it remains unsolved are structural, not just engineering laziness.

Rasterio, fiona, and pyogrio ship binary wheels that vendor their own copy of `libgdal` and its dependencies inside each wheel. This works because each package loads its own private copy of the library. The GDAL bindings can't do this, because if `pip install GDAL` delivered a wheel containing `libgdal.so`, and the user also had rasterio installed with its own `libgdal.so`, both copies would be loaded into the same Python process. The result is crashes or silent state corruption, depending on import order.

The problem is worse than just one duplicated library. The GDAL Python bindings consist of several separate extension modules (`osgeo.gdal`, `osgeo.ogr`, `osgeo.osr`, `osgeo.gdal_array`) that all need to link against the same `libgdal` instance and share global state — driver registration, error handlers, configuration options. If these were statically linked into separate `.so` files, each would get its own copy of GDAL's global state, which would be incorrect.

Even Rouault outlined two possible solutions: a single monolithic extension module (significant SWIG refactoring), or a dynamically linked `libgdal.so` with all public symbols renamed to avoid collision — "quite an adventure."

## Why R doesn't have this problem

The contrast with R is instructive and highlights how packaging model design choices cascade into ecosystem-wide consequences.

CRAN builds binaries centrally. When sf or terra is submitted, CRAN's build machines compile it against a known GDAL installation that CRAN maintains. On Windows this is Rtools with bundled libraries; on macOS it's the CRAN recipes infrastructure. Every R package that needs GDAL links against the same copy. There is no symbol collision because there is one `libgdal`, shared across sf, terra, vapour, gdalraster, and any other package.

R's linking model supports shared native dependencies. Multiple packages share a single dynamic `libgdal`. In Python, each wheel is a self-contained unit — pip has no concept of a shared native library dependency that multiple packages can link against at runtime. The Rtools toolchain bundles a curated C/C++ library stack that any R package can link against — functionally equivalent to what conda-forge provides for Python, but integrated into CRAN's official distribution workflow. There is no PyPI equivalent.

Python's philosophy of "every wheel ships everything it needs" works well for simple packages but creates duplication and collision problems for the geospatial stack, where multiple packages all require the same complex C library.

## Social barriers vs technical barriers

Both ecosystems have a dominant high-level package — sf in R, rasterio in Python — and in both cases the popularity of that package shapes how users think about GDAL access. But the nature of the constraint is fundamentally different.

In R, the barrier is social and psychological. When a new package like vapour or gdalraster appears and works directly with the GDAL C API, the community response is "how does this relate to sf?" — as though sf were the gatekeeping layer through which all GDAL work must pass. But this is a perception, not a technical reality. Because every R package links against the same shared `libgdal`, you can `library(sf)` and `library(gdalraster)` in the same session without conflict. New packages that expose different parts of GDAL coexist freely.

In Python, rasterio's dominance has created a technical barrier rather than merely a social one. If a user's environment is built around rasterio's PyPI wheels, adding `osgeo.gdal` risks crashes from duplicate `libgdal` symbols. The ecosystem has organised around rasterio's subset of GDAL not only by preference but because the wheel architecture forces a choice: use rasterio's vendored GDAL, or use the GDAL bindings, but combining both via pip requires conda-forge or a co-built wheel set.

This means that parts of GDAL not exposed by rasterio — the multidimensional API for NetCDF/HDF5/Zarr, advanced VRT operations, the full `gdal.Warp` option space — are not merely overlooked. They are structurally harder to access for users whose environments are built on PyPI wheels. The packaging architecture has, as an emergent consequence, narrowed the effective surface area of GDAL available to the average Python user.

## The practical landscape (2026)

Three options exist for binary GDAL Python installation:

**Christoph Gohlke's geospatial-wheels** (Windows only): co-built wheels for GDAL, rasterio, fiona, pyogrio, and others, all linked against the same library versions. The one pip-based environment where `import osgeo.gdal` and `import rasterio` safely coexist, because they share underlying libraries. Maintained by one individual.

**Petr Tsymbarovich's gdal-wheels** (Linux + Windows): uses cibuildwheel and vcpkg. Published to a GitLab PyPI registry. Cannot safely coexist with rasterio PyPI wheels — the symbol collision problem applies in full. Also maintained by one individual.

**conda-forge** (all platforms): packages `libgdal` as a shared library that all geospatial packages link against. This most closely mirrors R/CRAN's model and is the only approach that fully solves co-installability. The tradeoff is operating in conda's packaging ecosystem rather than pip's, and the risk that `pip install rasterio` inside a conda env reintroduces the duplication problem.

| | Gohlke wheels | gdal-wheels | conda-forge |
|---|---|---|---|
| Platforms | Windows | Linux + Windows | All |
| Safe with rasterio | Yes (co-built) | No | Yes (shared libgdal) |
| pip compatible | Yes (custom index) | Yes (GitLab index) | No (conda/pixi) |
| Governance | Individual | Individual | Community |

## What this tells us about ecosystem design

The GDAL wheel problem is a case study in how packaging architecture shapes what's possible. R's centralised build model — one shared library, CRAN compiles everything — eliminated the problem before it existed. Python's decentralised wheel model — each package ships its own dependencies — created a structural constraint that no amount of engineering effort has fully resolved.

The lesson isn't that one model is better. It's that the choice reverberates: R packages can freely expose different views of the same underlying library, while Python packages are pressured to be self-contained kingdoms. For a library as large and interconnected as GDAL, the kingdom model creates borders where none should exist.

<!--
## Notes (not for publication)

The full gdal-python-binaries.md has detailed installation commands for 
each option (pip, uv, conda, pixi), version-specific configuration for 
pyproject.toml, and links to the relevant gdal-dev mailing list threads.
Keep as reference material alongside this fragment.

The osgeo namespace discovery issue (tab completion doesn't work because 
SWIG doesn't wire up __all__ or lazy imports) is a nice detail for a 
sidebar but was cut from this fragment.

GDAL's multidimensional API (MDArray, Dimension, Group) being accessible 
ONLY through osgeo.gdal — not through rasterio, fiona, or pyogrio — is 
the key example of how the packaging constraint narrows GDAL's effective 
surface area. This connects to Ch 10 (Zarr, multidimensional data).
-->
