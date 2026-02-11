# Six string types and four ownership models

<!-- harvested from mdsumner/gdal-r-python cpp-strings-r-gdal.md -->
<!-- target: Appendix or Ch 11 sidebar -->
<!-- status: SNIPPET — full reference in cpp-strings-r-gdal.md -->

Writing R packages that wrap GDAL means working at the intersection of four memory management philosophies: C (a string is an address, ownership by loose agreement), C++ standard library (a string is a value type, RAII cleanup), R's C API (strings are garbage-collected, interned, immutable), and GDAL (C strings with its own allocation conventions — CSL lists, internal buffers, caller-owns vs callee-owns).

The six types you encounter are: `const char*` (raw C string — who owns this and when does it die?), `std::string` (C++ owned, the safe harbour for boundary crossings), `char**`/CSL (GDAL string lists — null-terminated array of null-terminated strings, the hand-rolled `std::vector<std::string>` from the C era), `STRSXP`/`CHARSXP` (R's interned immutable string pool), `Rcpp::CharacterVector` (convenient but indexing returns a proxy, not a string), and `cpp11::strings`/`cpp11::writable::strings` (read-only vs writable distinction explicit at the type level).

The practical core is the conversion matrix: almost everything copies. The only free conversions are views via `.c_str()` or `CHAR()`, both with lifetime constraints. Going into R always copies and interns. `std::vector<std::string>` is the safest staging container when shuffling many strings between systems.

The ownership traps are predictable but lethal. Dangling `.c_str()` from a temporary `std::string`. Holding GDAL internal pointers past the next API call. Forgetting `CSLDestroy()` on a list you built. Calling `CSLDestroy()` on `GDALGetMetadata()`'s internal storage. The mental model: every `const char*` is a question about who owns the memory and when it dies. Answer that before you use it.

The CLI-style argument tokenisation trap also catches everyone at least once: GDAL utility wrappers (`GDALWarp`, `GDALTranslate`) take `char**` as pre-tokenised argv-style arrays, not command-line strings that get shell-parsed. `"-ts 1024 0"` as one string won't work; it needs to be three separate elements `"-ts"`, `"1024"`, `"0"`. In R this is natural (character vectors are already tokenised), but the confusion comes from thinking of these as command lines rather than argument arrays.

On the Python side, the memory/lifetime problems disappear (GC handles it), but a different confusion emerges: there are five ways to pass options to GDAL (string lists, keyword arguments, options objects, dicts, JSON strings), and they don't always compose or interchange cleanly. When in doubt, use the string list form — it maps 1:1 to the C API and works everywhere.

<!--
## Notes (not for publication)

The full cpp-strings-r-gdal.md has the complete conversion matrix table, 
all five common patterns for GDAL wrappers (reading metadata, passing open 
options, std::vector staging area, config option RAII guard, CLI argument 
tokenisation), and all five ownership traps with code examples.

The CSLConstList migration in GDAL 3.x (from char** to const char* const*) 
is part of a wider const-correctness push that makes ownership clearer at 
the type level — worth mentioning when discussing GDAL's evolution.

cpp11's read-only vs writable distinction is a genuine improvement over Rcpp 
for clarity when wrapping GDAL functions. The book should recommend cpp11 
over Rcpp for new GDAL wrapper packages.
-->
