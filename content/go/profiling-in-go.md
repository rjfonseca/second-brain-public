---
type: tecnologia
description: "Anotações técnicas e referências sobre Profiling in Go."
tags:
  - notes/go
---

# Profiling in Go

## What is this?
Go has some good profiling tools already in the official packages.

See: [Profiling Go Programs - The Go Blog](https://blog.golang.org/profiling-go-programs)

## pprof
### Interpreting the call graph
- [Readme.Md (github.com)](https://github.com/google/pprof/blob/master/doc/README.md#interpreting-the-callgraph)
## Profile-Guided Optimization
Go allows for compiling optimizations using the results of a pprof.
- [Go Pgo (theyahya.com)](https://theyahya.com/posts/go-pgo/)
## External references
### Go Profiler Notes
This guy is compiling an extensive [documentation](https://github.com/DataDog/go-profiler-notes) on the subject.
### The Full Go Profiler
This [project](https://github.com/felixge/fgprof) aims to provide a CPU profiler that analyzes both On-CPU and Off-CPU time together.
fgprof is designed for analyzing applications with mixed I/O and CPU workloads. This kind of profiling is also known as wall-clock profiling.
