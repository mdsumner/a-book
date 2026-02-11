# Spatial Book: Fragment Manifest

<!-- Master index of all harvested fragments. Update as new material is processed. -->

## Naming convention

Files: `frag-{chapter}-{short-slug}.md`  
Example: `frag-ch08-geometry-is-tables.md`

## Status codes

- **DRAFT** — first pass, needs editing
- **SNIPPET** — small piece, may merge into a larger fragment
- **READY** — edited, can drop into chapter scaffold
- **MERGED** — already incorporated into a chapter draft

## Fragments

| File | Source | Target Chapter | Words | Status | Notes |
|------|--------|---------------|-------|--------|-------|
| frag-ch08-geometry-is-tables.md | table-r-book (01-overview, 02-GIS-contract, 03-examples, 04-shared-framework) | Ch 8: Geometry as topology | ~1800 | DRAFT | Five-form taxonomy, GIS contract, pipe-dream API, proto-silicate lineage |
| frag-ch08-3d-globe-story.md | rogue-book (02-poly3d) | Ch 8 or Ch 12: case study | ~600 | DRAFT | Why tables unlock dimensions; polygon→PSLG→triangulate→drape on sphere |
| frag-ch08-rosetta-stone.md | rogue-book (01-intro), updated | Ch 8: Geometry as topology | ~100 | SNIPPET | Entity terminology across sp/sf/ggplot2/terra/gdalraster/GDAL |
| frag-ch08-triangulation-taxonomy.md | rogue-book (09-triangulation, 02-poly3d) | Ch 8 or Ch 9 | ~250 | SNIPPET | Delaunay vs constrained Delaunay vs ear-clipping |
| frag-ch01-six-numbers.md | rw-book (georef.qmd, xarray_unref_matrix.ipynb) | Ch 1 + Ch 4 | ~1500 | DRAFT | Raw matrix → extent → geotransform → MEM DATAPOINTER → .bil/.hdr → GCPs → a_ullr |
| frag-ch04-grid-taxonomy.md | rw-book (grids.qmd), conversations with Ryan Abernathey | Ch 3 or Ch 4 | ~800 | DRAFT | A–H classification of grid referencing modes; xarray vs GDAL worldview |
| frag-ch04-ghrsst-float32.md | rw-book (ghrsst-netcdf.qmd) | Ch 3 or Ch 4 | ~700 | DRAFT | GHRSST broken grid, Float32 precision trap, VRT one-liner fix |
| frag-ch09-extraction-rasterization.md | rw-book (experiments/complement.Rmd) | Ch 9: Grids as sparse structures | ~250 | SNIPPET | Extraction/rasterization as complements; standalone until gridburn re-harvest |
| frag-ch01-centre-vs-edge.md | rw-book (r-image.qmd) | Ch 1: What is a grid? | ~200 | SNIPPET | image() vs contour() vs rasterImage(); the half-pixel problem everywhere |
| frag-ch11-notebook-pain.md | rw-book (georef.qmd + ipynb coexistence) | Ch 7 or Ch 11 | ~200 | SNIPPET | Cross-language document formats; quarto vs ipynb limitations |
| frag-ch04-tiepoints-formats-carry.md | gdal-r-python (georef-tiepoint/raster-coordinate.md + tiepoint-illustrate-expanded.md) | Ch 4: How formats encode "where" | ~2000 | DRAFT | Tiepoint lineage, four georef models, two-worlds thesis, "formats carry / libraries warp", GeoZarr critique. Ryan Abernathey conversations shaped two-worlds framing |
| frag-ch11-wheel-problem.md | gdal-r-python (installation/gdal-python-binaries.md) | Ch 11: The installation problem | ~1200 | DRAFT | Why pip install GDAL is broken; symbol collision; R vs Python packaging as structural analysis; social vs technical barriers |
| frag-ch07-parallel-gdal.md | gdal-r-python (parallel-gdal-r-python.md) | Ch 7: Same problem, different clothes | ~900 | DRAFT | GIL mechanics for GDAL, threading vs process parallelism, Python ThreadPoolExecutor vs R mirai comparison |
| frag-ch11-cpp-strings.md | gdal-r-python (cpp-strings-r-gdal.md) | Appendix or Ch 11 sidebar | ~500 | SNIPPET | Six string types, ownership model, conversion matrix, tokenisation trap. Full reference kept in source |
| frag-ch03-gis-origin-story.md | silicate proto-book (sc-book: 01-intro.Rmd, 02-limitations.Rmd) | Ch 3: What does GDAL do? (or intro) | ~600 | DRAFT | Arc-node→path compromise, ArcView/shapefile, sf limitations (three structural), topology built-and-discarded pattern |
| frag-ch02-crs-practical-problem.md | antequated.org (polar-maps-now 2017, map-projections 2023, SOmap 2019) | Ch 2: What is a CRS? | ~1100 | DRAFT | Four ingredients, three tool types (reproject/reproject/warp), projection families vs parameters, real-world Antarctic projection catalog, chicken-egg of extent/projection, Tissot link |
| frag-ch02-crs-opening-draft.md | brainstorm session 2026-02-11 | Ch 2: What is a CRS? (chapter opening) | ~2500 | RAW DRAFT | Unpacks CRS into family/centre/params/datum; developable surfaces demystified; Mercator→TM→UTM as "one family, sixty centres"; CRS is not a map (need extent too); zero-centre trick: family+centre+distance=map, dissolves chicken-egg problem; rasterization is projection into index space; warp-the-grid flips conventional wisdom; "when geometry is hard, check the coordinate system." Needs figures. |
| frag-ch02-equal-area-warp-stats.md | brainstorm session 2026-02-11 | Ch 2 (callback) / Ch 9 / Ch 12 | ~650 | RAW FRAGMENT | OISST weighted mean three ways: manual R weights, xarray weighted, warp to LAEA and plain mean — exact match. Proves warp-the-grid applies to statistics not just geometry. "The projection carries meaning." Connects Ch 2 CRS principles to Ch 4 grid taxonomy and Ch 9 extraction. |

## Pending sources (not yet processed)

| Source | Status | Notes |
|--------|--------|-------|
| rw-book repo | HARVESTED | 6 fragments extracted; repo archived |
| xarray_unref_matrix.ipynb | HARVESTED | Content merged into frag-ch01-six-numbers.md |
| mdsumner/gdal-r-python | HARVESTED | 4 fragments + 1 snippet extracted from 12 files (see reference inventory below) |
| mdsumner/sc-book (silicate proto-book) | HARVESTED | 1 fragment (GIS origin story) from 6 Rmd files; bulk is redundant with frag-ch08-geometry-is-tables.md (see reference inventory below) |
| rbind/antequated.org | HARVESTED | 1 fragment (CRS practical problem) from 3 posts; Southern Ocean map tutorial kept as reference (see reference inventory below) |
| mdsumner/polar-mapping-oghub | NOT INSPECTED | |
| hypertidy.org blog posts (17+) | NOT INSPECTED | |
| Past conversations: tiepoints/CRS | FOLDED IN | Tiepoints essay was harvested into gdal-r-python, now frag-ch04-tiepoints-formats-carry.md |
| Past conversations: rasterio Affine | NOT EXTRACTED | Ch 7 material |
| Past conversations: abstraction addiction | NOT EXTRACTED | Ch 6 material |
| Past conversations: vertex pool/silicate | NOT EXTRACTED | Ch 8 material |
| Past conversations: VRT/GTI | NOT EXTRACTED | Ch 10 material |
| Past conversations: vaster blog draft | NOT EXTRACTED | Ch 1 material |
| gridburn docs-design (6 docs) | UPLOADED, NOT HARVESTED | Ch 9 material; will merge with frag-ch09-extraction-rasterization.md |

## gdal-r-python reference inventory

Files NOT harvested into fragments but tracked for reference:

| File | Why kept as reference | Potential use |
|------|----------------------|---------------|
| geometry-is-tables.md | Already harvested as frag-ch08 (duplicate) | — |
| tiepoint-illustrate.md | Superseded by expanded version | — |
| tiepoint-illustrate-expanded.md | Visual code companion; key content in frag-ch04-tiepoints-formats-carry.md | Ch 4 figures (R + Python comparison plots) |
| raster-coordinate.md (full) | Full 7000-word essay with appendices; fragment is the distilled argument | Ch 4 deep-dive, GeoZarr appendix, open questions for gdal-dev |
| cpp-strings-r-gdal.md (full) | Complete conversion matrix + 5 patterns + 5 traps; snippet is the overview | Appendix reference table |
| gdal-python-binaries.md (full) | Installation commands, uv/pixi config, mailing list citations | Ch 11 practical section |
| parallel-gdal-r-python.md (full) | Complete code examples, benchmark numbers from pixtract | Ch 7 worked examples |
| gha-fixes/cheatsheet.md | Pure howto; no conceptual payload for book | Blog post or standalone reference |
| GNM/gdal-gnm-guide.md | Niche GDAL feature nobody knows about | Possible Ch 3 sidebar ("GDAL features you didn't know existed") |
| positron/orphaned-positron-guide.md | Pure howto for specific tool | Blog post or standalone reference |
| zarr-examples/public-zarr-sources.md | Catalog of public Zarr test sources | Ch 10 appendix; testing reference |
| zarr-examples/README.md | Link collection to attempted Zarr projects | Historical reference |

## sc-book (silicate proto-book) reference inventory

The silicate proto-book (~2017, bookdown format) was the first extended write-up of the PATH/PRIMITIVE decomposition. Most conceptual content is already captured in frag-ch08-geometry-is-tables.md. One new fragment harvested (GIS origin story → Ch 3).

| File | Why kept as reference | Potential use |
|------|----------------------|---------------|
| 01-intro.Rmd | GIS origin narrative harvested into frag-ch03; aspirational feature list is historical | Ch 3 fragment source; package lineage reference (spbabel → gris → rangl → silicate) |
| 02-limitations.Rmd | sf structural limitations + shared-edge demo code; argument already in frag-ch08 | Ch 8 worked examples (get_shared_edge function, nc.shp topology demo) |
| 03-normalization.Rmd | PATH/PRIMITIVE table counts, segment model, raster sidebar; argument in frag-ch08, raster in Ch 1/Ch 4 | Ch 8 worked examples; raster taxonomy (affine/rectilinear/curvilinear) already covered |
| 04-translator.Rmd | Concrete conversions: sf→base plot, sf→spatstat, sf→rgl, constrained triangulation | Ch 8 worked examples; spatstat/rgl translator code |
| 05-extras.Rmd | Grant proposal framing; repeated motivations from 01/02 | Historical reference only |
| index.Rmd | One-paragraph preamble ("removal of topology has been overly destructive") | Possible book epigraph |

## antequated.org reference inventory

Blogdown site on rbind org for polar mapping techniques. ~90% of uploaded content was Hugo theme boilerplate (ghostwriter, future-imperfect, lithium-theme examples) — all skipped. One fragment harvested combining best of three posts into Ch 2 material.

| File | Why kept as reference | Potential use |
|------|----------------------|---------------|
| 2017-06-14-general-southern-ocean-map.Rmd (Cleeland) | Worked example: full Southern Ocean map with sp/rgdal/raster, graticules, CCAMLR polygons, fronts, labels | Ch 2 or Ch 12 worked example; historical snapshot of sp-era workflow |
| 2017-06-14-polar-maps-now.Rmd | Pedagogical framing harvested into frag-ch02; code examples using sp/rgdal | Ch 2 code examples (sp era) |
| 2019-03-20-get-antequated-with-somap.Rmd | SOmap package demo, auto-projection | Ch 2 code examples (SOmap); potential Ch 12 pattern |
| 2023-04-04-map-projections.qmd | Most mature version; projection catalog and three-tool-types harvested into frag-ch02 | Ch 2 code examples (terra era); Mercator vs UTM cellSize demo |
| content/about.md | One-liner ("antequated philosophy rails against tropical-bias") | Possible book epigraph |

## Cross-reference notes

Small snippets that should be folded INTO existing fragments rather than kept standalone:

- ~~**"sp draws every object separately because polygon() only does one colour"** (rogue-book polys-in-R) → fold into frag-ch08-geometry-is-tables.md~~ DONE
- ~~**"Manifold drawings hold points/lines/areas in one layer"** (rogue-book 04-manifold) → fold into frag-ch08-geometry-is-tables.md~~ DONE
- **"gris joinable chain" description** already covered in frag-ch08-geometry-is-tables.md lineage notes

## Chapter outline (for reference)

- Ch 1: What is a grid?
- Ch 2: What is a CRS?
- Ch 3: What does GDAL do?
- Ch 4: How formats encode "where" (tiepoints essay)
- Ch 5: How R thinks about spatial
- Ch 6: How Python thinks about spatial
- Ch 7: Same problem, different clothes
- Ch 8: Geometry as topology
- Ch 9: Grids as sparse structures
- Ch 10: Virtual composition and lazy evaluation
- Ch 11: The installation problem
- Ch 12: Patterns for real work
- Ch 13: What's changing
