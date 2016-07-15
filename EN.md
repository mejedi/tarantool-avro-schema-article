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

## The Problem

We were tasked with implementing a JSON storage on top of Tarantool for a
major cellular operator.
The structure of a JSON document was fully prescribed by a schema.
A document had to be validated before insertion into the database.
There was a requirement to drop some bits prior to insert, like key names, in order to optimize data footprint.
These bits were restored later using a schema.
Finally, there was a catch: a schema will evolve in time, so it is often necessary
to convert data between schema revisions on the fly.

Due to the high performance requirements, JIT compilation was deemed necessary.

Though the developed solution is highly specific, the techniques we've learned
are applicable to a wider range of tasks, like indexing and constraints enforcement
on a semi-structured data.
These are exciting directions, since we are actively working on document
features in Tarantool.

