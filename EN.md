# Accelerating Data Processing in a Database with JIT&nbsp;Compilation: Lessons Learned

Technology landscape is ever changing.
Modern databases are very different from the 1970-s classics,
due to such tectonic shifts as SSD drives arriving to the market,
RAM prices dropping drastically
and distributed computing becoming commonplace.
We beleive there is another change currently underway.

JIT compilation was once a domain reserved for the die-hard compiler experts.
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
to make some data processing tasks in Tarantool faster.

## About Tarantool

Tarantool is a modern multi-paradigm database.

`[TODO: extend this section, upto 3 statements]`

## The Problem

Recently we've built a JSON storage on top of Tarantool for a
major cellular operator. Incoming data is validated against a schema.
The set of schemas may evolve in time.
Assume the storage has an entity in schema **A** but a client wants it in schema **B**.
Converting between different schema revisions transparently is a key feature.

Consider a simple JSON document below:

```json
{
  "FirstName": "John",
  "LastName":  "Doe"
}
```

The document is validated against a schema (we've adopted Apache Avro notation):

```json
{
    "name": "Person",
    "type": "record",
    "fields": [
        { "name": "FirstName", "type": "string" },
        { "name": "LastName",  "type": "string" }
    ]
}
```

A process we call *flatten* yields a compact tuple.
This tuple goes into storage.
Omiting key names significantly reduces data footprint.
Names are recovered from the schema when necessary.

```json
[ "John", "Doe" ]
```

Row format in Tarantool is flexible:
the number of fields in a row and individual field types may vary from row to row.
Only those fields participating in indexes have type restrictions attached.
This flexible data model makes it possible to store entities in different schema
revisions side by side.

Implementing *flatten* and the inverse transformation is challenging.
The routines convert data between different schema revisions on the fly.
Another tricky feature is *field defaults*: if a field was omitted, a default value from schema definition is used.

Initially the project used the stock Apache Avro C library. The library implements a generic data handling machinery which consults schema definition in runtime, very similar to a programming language's interpretor. Unfortunately, the performance was unsatisfactory, which could be attributed to frequent memory allocation, lots of indirection and the interpretor's overheads.

We explored code generation next. For each schema, the specialised code implementing data transformations was generated. These efforts plus designing an efficient data handling runtime resulted in a major speedup. The generator was written in Lua and produced Lua code. Dynamic languages make it realy easy to generate code in runtime! Since the output was a human-readable Lua code, debugging was relatively easy. Finally we implemented a new backend in Terra (Lua dialect) targeting LLVM and got yet another impressive speedup. See the following sections for more details.

![Performance during project's lifetime](https://github.com/mejedi/tarantool-avro-schema-article/blob/master/avro_perf.png?raw=true)

The version employing LLVM exceeded 3M *flatten* OPs/sec on a single core of a circa 2014 MacBook Pro (2.2 GHz Core i7 processor). The benchmark used a somewhat simple schema with 14 fields, 2 nested records plus a 4 element string array. Performance scales almost lineary with a schema complexity. It's not uncommon to encounter a 140 field schema in the wild. With such a complex schema the projected performance is 300K OPs/sec. Assuming that only 10% of the running time is alloted for data processing and the remaining 90% are spent doing other things, it translates into 30K OPs/sec.

It means that the project ultimately suceeded, exceeding its performance goals of 10K OPs/sec on a single core. We'd also like to share our experience of leveraging JIT-compilation techniques in terms of development costs, check out *Conslusion* section.

## Apache Avro Schema

`[Plan: About Avro. Type system. Mapping schema versions.]`

## Data Transformation: Execution Model

`[Plan: DOM vs SAX. Lightweight DOM (no pointers). Pocedural validation. Output. Schema mapping done in compile time. Reporting errors.]`

## LuaJIT

`[Plan: Dynamic languages make it easy to generate code in runtime. FFI extension to Lua. A stepping stone to JIT compilation.]`

## Conclusion

`[...]`

Though the developed solution is highly specific, the techniques we've learned
are applicable to a wider range of tasks, including constraints enforcement
and indexing of a semi-structured data.
These are exciting directions, since we are actively working on document
features and SQL in Tarantool.
