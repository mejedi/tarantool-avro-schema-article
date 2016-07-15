# Accelerating Data Processing in a Database with JIT&nbsp;Compilation: Lessons Learned

The technology landscape is ever changing.
Modern databases are very different from the 1970-s classics,
due to such tectonic shifts as SSD drives arriving to the market,
RAM prices dropping drastically
and distributed computing becoming commonplace.

We beleive there is another change currently underway.
JIT compilation is already tractable by regular engineers, not only die-hard compiler experts.
Libraries developed by LLVM project made it possible to
generate code in an architecture-independent Internal Reperesentation (IR) programmatically.
The IR is processed by a highly-sophisticated optimizing compiler, yeilding
machine code for any major architecture, including `x86_64`, `arm` and `aarch64`.

Traditionally, query interpretor overheads were dwarfed by slow disk access times.
However nowdays, with the vast amounts of RAM available, interpretor overheads
are no longer negligible.
JIT compilation is a widely used technique to improve interpretors' performance.

We'd like to share our experience of leveraging JIT compilation
to improve performance of some data processing tasks in Tarantool.
