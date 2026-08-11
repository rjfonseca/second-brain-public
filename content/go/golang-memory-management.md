---
type: tecnologia
description: "Go runtime manages memory allocation in a very specific way. Go uses Garbage Collection to clear out the heap."
tags:
  - notes/go
---

# Golang Memory Management

## What is this?
Go runtime manages memory allocation in a very specific way.

Go uses *Garbage Collection* to clear out the heap.
### The go runtime
Go creates a Logical Processor (P) and a Machine(M) for each OS thread and one OS thread per core.

[https://i.imgur.com/wThLAbQ.png](https://i.imgur.com/wThLAbQ.png)

```go
// NumCPU returns the number of logical
// CPUs usable by the current process.
fmt.Println("Logial CPUs:", runtime.NumCPU())
```

```text
Logial CPUs: 8
```

### The virtual memory
[https://i.imgur.com/vFtq3uj.png](https://i.imgur.com/vFtq3uj.png)

- `mheap`: THE heap. This is the main block of memory and where the garbage collector works;
- `arena`: Chunks of virtual memory of 64MB. Maps pages to spans;
- `mcentral`: Groups spans of the same size. Requests memory to `mheap`. Contains two doubled-linked lists:
  1. `empty`: Spans with no free objects of spans cached in `mcache`; and
  2. `non-empty`: spans with free objects
- `mcache`: Used by a Logical Processor to store small objects (<=32Kb); and
- `mspan`:
  [https://i.imgur.com/IxjG2aF.png](https://i.imgur.com/IxjG2aF.png)

### Go's garbage collection
Go's garbage collection works as follows:

1. Stop the world: lock the heap, non GC go-routines are stopped;
2. Concurrent Marking: Paint heap objects as *alive*;
3. Unstop the world: unlock the heap, start cleaning processes; than
4. On new allocation, reclaim memory *not alive*;

See [this slides](https://speakerdeck.com/deepu105/go-gc-visualized) to have a better idea.

- [A Guide to the Go Garbage Collector - The Go Programming Language](https://go.dev/doc/gc-guide)
## Why this is important?
The number of pages are limited (per size) and go doesn't optimize struct sizes. There could be waste of memory and also stress the memory management by allocating.
We can manually reorganize struct fields to reduce the struct size.

```go
type Foo struct {
	aaa [2]bool // offset of bytes: 0
	bbb int32   // offset of bytes: 4
	ccc [2]bool // offset of bytes: 8
}

type Bar struct {
	aaa [2]bool // offset of bytes: 0
	ccc [2]bool // offset of bytes: 2
	bbb int32   // offset of bytes: 4
}

ff := Foo{}
bb := Bar{}
fmt.Printf("offsets of fields: aaa: %+v; bbb: %+v; ccc: %+v\n", unsafe.Offsetof(ff.aaa), unsafe.Offsetof(ff.bbb), unsafe.Offsetof(ff.ccc))
fmt.Printf("offsets of fields: aaa: %+v; ccc: %+v; bbb: %+v\n", unsafe.Offsetof(bb.aaa), unsafe.Offsetof(bb.ccc), unsafe.Offsetof(bb.bbb))
fmt.Printf("required alignment: %+v\n", unsafe.Alignof(Foo{})) // required alignment: 4
fmt.Printf("required alignment: %+v\n", unsafe.Alignof(Bar{})) // required alignment: 8
```

```text
offsets of fields: aaa: 0; bbb: 4; ccc: 8
offsets of fields: aaa: 0; ccc: 2; bbb: 4
required alignment: 4
required alignment: 4
```

Also understanding where the variable will be allocated (stack or heap) and how can help speed up de application by avoiding unnecessary garbage collection. The escape analysis (link here latter) can help in this subject.

Discord switch from go to rust because of that.

There is a linter project that checks and fixes this: [GitHub - dkorunic/betteralign: Make your Go programs use less memory (maybe)](https://github.com/dkorunic/betteralign).

## Resources
- [Visualizing memory management in Golang](https://deepu.tech/memory-management-in-golang/)
- [Scheduling In Go : Part II - Go Scheduler](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html)
- [Manual Memory Management in Go using jemalloc](https://dgraph.io/blog/post/manual-memory-management-golang-jemalloc/)
- [Why Discord is switching from Go to Rust](https://blog.discord.com/why-discord-is-switching-from-go-to-rust-a190bbca2b1f)
- [Padding is hard](https://dave.cheney.net/2015/10/09/padding-is-hard)
- [Structure size optimization in Golang (alignment/padding). More effective memory layout (linters).](https://itnext.io/structure-size-optimization-in-golang-alignment-padding-more-effective-memory-layout-linters-fffdcba27c61)
- [A visual guide to Go Memory Allocator from scratch](https://medium.com/@ankur_anand/a-visual-guide-to-golang-memory-allocator-from-ground-up-e132258453ed)
- [The Go Memory Model - The Go Programming Language](https://tip.golang.org/ref/mem)
