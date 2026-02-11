# Equal-area warp as weighted mean

<!-- source: brainstorm session 2026-02-11 -->
<!-- chapter: Ch 2 (callback), or Ch 9/Ch 12 (patterns) -->
<!-- status: RAW FRAGMENT — needs code examples fleshed out -->

## The problem

Calculate the mean equatorial/subtropical/temperate sea surface temperature (SST) every day from global OISST data. OISST is a regular longlat grid — quarter-degree cells. But cells near the equator cover far more area than cells near the poles. A naive `mean()` over the raw grid over-represents high latitudes. The correct answer requires an area-weighted mean, where the weight is proportional to cos(latitude).

## Three approaches

**1. Manual weighted mean in R.** Compute cos(latitude) for each cell, construct a weight matrix, multiply, sum, divide. The code isn't complicated but it's confusing — you're managing array dimensions, broadcasting weights against a daily time series, making sure you've capped the latitude range correctly, handling NA ocean cells. Easy to get subtly wrong.

**2. xarray weighted mean in Python.** xarray has built-in support for weighted operations. You construct latitude weights with `np.cos(np.deg2rad(ds.lat))`, pass them to `ds.weighted()`, and call `.mean()`. Elegant, readable, correct. But you still need to know that longlat cells aren't equal area, know to ask for weights, and know to use cosine of latitude specifically.

**3. Warp to LAEA and take a plain mean.** Create a Lambert Azimuthal Equal Area grid centred on null island (0°N, 0°E) with a cell size of ~25 km — roughly matching OISST's equatorial resolution. Warp the OISST data to that grid. Take `mean()`. No weights.

The result matches the weighted calculation exactly.

## Why it works

An equal-area projection, by definition, preserves area. Every cell in the warped grid covers the same area on the Earth's surface. A simple unweighted mean over equal-area cells *is* an area-weighted mean — the weighting is built into the grid geometry. You didn't compute weights; you changed the coordinate system so that weights aren't needed.

This is the same principle as the antimeridian example: when the calculation is hard, check whether it's hard because of the coordinate system. Longlat cells are not equal area, so statistics over longlat grids require correction. Equal-area cells are equal area, so statistics are just... statistics.

## What this demonstrates

- **The projection carries meaning.** Choosing LAEA isn't just a display decision — it changes what `mean()` computes. The CRS is part of the analysis, not just the visualisation.
- **Warp-the-grid generalises.** It's not just for avoiding geometry bugs (antimeridian). It simplifies statistical operations by encoding the right geometric properties into the grid itself.
- **The "right" tool depends on what you see.** The R approach requires you to understand the weighting problem and implement it. The xarray approach requires you to understand the weighting problem and use the right API. The warp approach requires you to understand that the problem *is* the coordinate system — and then the solution is trivial.
- **Connection to rasterization-as-projection.** Warping the grid is rasterizing the data onto a new index space — one where the cells have the geometric property (equal area) that makes your statistic correct without adjustment.

<!-- 
## Notes

- The LAEA grid was centred on null island (0, 0) with extent roughly ±25000 km, 
  matching OISST's ~25km equatorial cell size
- Latitude range was capped (abs(lat) < some threshold) for the subtropical/temperate 
  calculation — this is a crop in longlat before warp, or equivalently a clip in 
  the LAEA grid after warp
- The match was "absolutely bang on" — not approximate, not close, exact to 
  floating point. This is because the cos(lat) weighting and the equal-area 
  cell area are mathematically the same correction.
- Could flesh this out with actual code showing all three approaches side by side
- The xarray version is a good contrast: it's clean, Pythonic, correct — but it 
  still treats the coordinate system as given and adds weights after the fact. 
  The warp version changes the coordinate system so the weights are unnecessary.
- This example might work best as a callback in Ch 9 or Ch 12, harking back to 
  Ch 2's "when the math is hard, check the coordinate system" principle
- Also connects to Ch 4 (grid taxonomy): OISST is rectilinear in longlat, 
  the warped version is regular in LAEA — the grid taxonomy matters for 
  what statistics you can compute naively
-->
