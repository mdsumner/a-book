# a-book

A working draft of a book about spatial data: grids, coordinate reference
systems, GDAL, and geometry, seen from R and Python at the same time.

## The thesis

Most spatial books teach a library. This one teaches the data. There are only a
handful of ideas in spatial computing, and they wear different costumes in every
tool, format and domain. Once you see the ideas, the tools become interchangeable.

Three threads run through every chapter:

1. **Projection is the universal verb.** Geographic to projected coordinates is a
   projection. Projected coordinates to grid index space is rasterization. Index
   space back to coordinates is extraction. The affine transform is the bridge.
   These are one operation applied at different stages of the pipeline.
2. **Representation is the problem.** Antimeridian wrapping, polygon splitting,
   topology that is built and then discarded, cos(latitude) weights: these are
   artefacts of the coordinate system or data structure, not of the geometry.
   Change the representation and the problem goes away.
3. **Decomposition reveals choices.** A CRS is a family, a centre and some
   parameters. A grid is six numbers. A geometry is one of five table forms. The
   standard tooling hands you an opaque bundle; the insight comes from pulling it
   apart into independent decisions.

## How this repo is organised

- `frag-chNN-slug.md` are harvested fragments, the raw material. Each carries a
  comment header with its source, target chapter and status. They are tracked in
  [MANIFEST.md](MANIFEST.md), which is the index of what exists, where it came
  from, and what is still to be harvested.
- `NN-slug.qmd` are the chapters. A chapter is a thin Quarto file that includes
  its fragments in order, with connective prose added as the chapter matures.
  Fragment numbering in filenames follows the original 13-chapter outline in the
  manifest; the chapter files follow the shorter first-edition outline below.
- `_quarto.yml` builds the book. `quarto render` produces `_book/`.
- `notes/` holds reviews and reference transcripts that are not book text.

## First-edition outline

Part I: Where

1. What is a grid?
2. What is a CRS?
3. How formats encode "where"

Part II: The engine

4. What GDAL does
5. Same problem, different clothes

Part III: Shape

6. Geometry as topology
7. Extraction and rasterization

Part IV: Ecosystem

8. The installation problem

Chapters on virtual composition, patterns for real work and what is changing are
deferred to a second pass; their material accrues in the manifest's pending list.

## Working on it

```
quarto render          # build the HTML book into _book/
quarto preview         # live preview
```

Fragments move through RAW, SNIPPET, DRAFT, READY and MERGED (see the manifest).
When a fragment reaches READY, demote its headings by one level and fold it into
the chapter file; until then the chapter includes it as is.

Content is ASCII only.
