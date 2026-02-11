# Parallelism: same problem, different runtimes

<!-- harvested from mdsumner/gdal-r-python parallel-gdal-r-python.md -->
<!-- target: Ch 7 (Same problem, different clothes) -->
<!-- status: DRAFT -->

## The bottleneck

Raster point extraction — and most GDAL workloads — is I/O bound. Reading and decompressing tiles from disk or network is the bottleneck, not the arithmetic. Parallelizing tile reads is the obvious win, but Python and R arrive at it through completely different runtime models, and the mechanics shape what's natural in each language.

## Python: threading works because of C

Python's Global Interpreter Lock (GIL) blocks concurrent Python bytecode execution, but GDAL's `ReadAsArray` drops into C code for the actual I/O and decompression. During that C call, the GIL is released. Multiple threads can do genuinely concurrent GDAL reads — the same reason numpy operations can be threaded.

`concurrent.futures.ThreadPoolExecutor` is the natural fit. Each thread opens its own `gdal.Open()` handle. For `/vsicurl/` sources this means parallel HTTP connections, which is exactly what you want. The numpy grouping code (argsort, split, fancy indexing) remains GIL-bound, but it runs before the parallel section — threads only do GDAL reads and array assignment.

Threading helps most for network I/O (`/vsicurl/`, `/vsis3/`) where threads wait on HTTP responses, and for compressed tiles (DEFLATE, ZSTD, LZ4) where decompression happens in C with the GIL released. On local SSD with uncompressed data, serial and parallel are barely different — thread overhead eats the gains.

Orthogonally, GDAL's internal `GDAL_NUM_THREADS` config option controls multi-threaded decompression per tile. Application-level thread pools and GDAL-internal thread pools can compound: eight threads each doing a tile read where GDAL internally uses four threads to decompress. Whether this helps or thrashes depends on the machine.

## R: processes work because there's no alternative

R has no GIL, but it also has no true threading for R code. All parallelism in R is process-based — you're forking or spawning separate R sessions. The question is how much friction that involves.

`mirai` with persistent daemons is the current best option for GDAL workloads. Each daemon is a separate R process that opens its own GDAL handles. No forking, no shared state, no surprises. The serialisation cost for passing data between processes is small for tile-sized payloads — a few thousand point coordinates plus a bounding box.

The fork-based approaches (`future::plan(multicore)`, `parallel::mclapply`) are risky with GDAL because `/vsicurl/` maintains HTTP connection pools and internal caches that don't survive fork cleanly. You can get corrupted reads, segfaults, or silent wrong results. Forking is fast (copy-on-write memory) but unsafe for anything involving network state or file handles.

## The structural comparison

| Dimension | Python (ThreadPoolExecutor) | R (mirai) |
|---|---|---|
| Mechanism | Threads (shared memory) | Processes (separate memory) |
| GDAL handle per worker | Open per thread | Open per daemon |
| Data sharing | Direct (numpy array) | Serialised (automatic) |
| Startup cost | Negligible | Once (daemons persist) |
| Memory overhead | Low (shared address space) | Higher (N × R sessions) |
| CPU parallelism for interpreted code | No (GIL) | Yes (separate processes) |
| CPU parallelism for C code (GDAL) | Yes (GIL released) | Yes (separate processes) |
| Risk of shared-state bugs | Dataset handles must be per-thread | None (fully isolated) |

Python threading works for GDAL because the GIL is released during C calls — threads share memory cheaply but get true parallelism for I/O. The tradeoff is that any Python-level computation remains single-threaded.

R's process-based parallelism is heavier (each worker is a full R session) but simpler to reason about — complete isolation, no shared state, no GIL. The tradeoff is serialisation overhead for passing data to and from workers.

For tile-grouped raster extraction, both approaches work well because the bottleneck is I/O, the per-task payload is small, and the expensive work (GDAL reads) happens in C regardless. The language runtime differences matter less than you'd expect once the workload is I/O-dominated.

## GDAL-specific tuning (both languages)

A few config options matter for parallel workloads:

`CPL_VSIL_CURL_CACHE_SIZE` (default 16 MB) — increase for workloads that revisit nearby tiles. For continent-spanning traverses, cache doesn't help; better to sort points by tile and process each tile once.

`GDAL_HTTP_MAX_RETRY` / `GDAL_HTTP_RETRY_DELAY` — set for network benchmarks to reduce jitter from transient S3 slowdowns.

`CPL_VSIL_CURL_ALLOWED_EXTENSIONS=.tif,.vrt` — prevents GDAL from probing for sidecar files (`.aux.xml`, `.ovr`) which adds extra HTTP requests on first open.

These apply identically in both languages because they're GDAL configuration, not language configuration. The runtime difference is in how you dispatch the work; the I/O tuning is the same.

<!--
## Notes (not for publication)

The full parallel-gdal-r-python.md has complete code examples for both 
languages including ThreadPoolExecutor setup, mirai daemon configuration, 
and benchmark numbers from pixtract development. Keep as reference.

The asyncio option (for truly massive COG workloads with thousands of 
HTTP range requests) is mentioned but not implemented in pixtract. 
Worth revisiting if the book covers cloud-optimised access patterns in depth.

For R, furrr + multisession is the predecessor to mirai and still commonly 
used. It's safer than multicore (no forking) but has more overhead than 
mirai's persistent daemons.

GDAL_NUM_THREADS interacts with application-level parallelism in non-obvious 
ways. 8 application threads × 4 GDAL decompression threads = 32 concurrent 
threads, which may or may not be what you want depending on core count and 
whether the bottleneck is I/O or CPU.
-->
