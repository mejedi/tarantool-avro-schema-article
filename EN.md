# Accelerating Data Processing in a Database with JIT&nbsp;Compilation: Lessons Learned

Technology landscape is ever changing.
Modern databases are very different from the 1970-s classics,
due to such tectonic shifts as SSD drives arriving to the market,
RAM prices dropping drastically
and distributed computing becoming commonplace.
We beleive there is another change currently underway.

JIT compilation was a domain reserved for the die-hard compiler experts.
LLVM project made it tractable for regular engineers.
The project provides convenient tools to
generate code in an architecture-independent Internal Reperesentation (IR) programmatically.
IR is processed by a highly-sophisticated optimizing compiler, yeilding
machine code for any major architecture, including `x86_64`, `arm` and `aarch64`.

Traditionally, query interpretor overheads were dwarfed by slow disk access times.
But nowdays, with the vast amounts of RAM available, interpretor overheads
became significant.
JIT compilation is a widely used technique to improve interpretors' performance.
We'd like to share our experience of leveraging JIT compilation
to improve performance of some data processing tasks in Tarantool.

## The Problem

We were tasked with building a JSON storage on top of Tarantool for a
major cellular operator.
The structure of a JSON document is fully prescribed by a schema.
A document is validated before insertion into the database.
Some bits, like key names, are dropped prior to insert in order to optimize data footprint.
These bits are restored later using a schema.
Finally, there is a catch: a schema will evolve in time, so it is often necessary
to convert data between schema revisions on the fly.

Due to the need for high performance, JIT compilation was deemed necessary.

Though the developed solution is highly specific, the techniques we've learned
are applicable to a wider range of tasks, including constraints enforcement
and indexing of a semi-structured data.
These are exciting directions, since we are actively working on document
features in Tarantool.
