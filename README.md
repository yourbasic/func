# Your basic func [sketch]

### Building graphs from functions only

#### Table of contents

[Introduction](#introduction)  
[API](#api)  
[Implementation](#implementation)  
[Testing](#testing)  
[Optimization](#optimization)

# Introduction

This text is about a tool for building **virtual graphs**.

The library is primarily based on functions,
all data structures in the API are immutable and
the main data structure `build.Virtual` is implemented
as a `struct` containing five `func` pointers.

The text comes with two [Go][golang] packages:

- [github.com/yourbasic/graph][graph]
  is a library of basic graph algorithms and
- [github.com/yourbasic/graph/build][graphbuild]
  is the tool for building virtual graphs.

### Petersen graph

Let's start with an example, the Petersen graph:

![Petersen graph](res/petersen.png)

*Picture from [Wikipedia][wikipetersen], [CC BY-SA 3.0][ccbysa30].*

To describe this graph in your typical graph library,
you need to somehow enter the edges  
{0, 1}, {0, 4}, {0, 5}, {1, 2}, {1, 6}, {2, 3}, {2, 7}, {3, 4},
{3, 8}, {4, 9}, {5, 7}, {5, 8}, {6, 8}, {6, 9}, and {7, 9}.

If you ask a mathematcian to describe the Petersen graph,
her answer would clearly be very different. Perhaps she would
say that the Petersen graph is most commonly drawn as a pentagon
with a pentagram inside, with five spokes.

```go
pentagon := build.Cycle(5)
pentagram := pentagon.Complement()
petersen := pentagon.Match(pentagram, build.AllEdges())
```
*Example from [godoc.org/github.com/yourbasic/graph/build][graphbuilddoc].*

Virtual graphs are constructed by **composing** and **filtering**
a set of **standard graphs**, or by writing
**functions that describe the edges** of a graph. 


## API

## Implementation

```go
type Virtual struct {
	// The `order` field is, in fact, a constant function.
	// It returns the number of vertices in the graph.
	order int

	// The `edge` and `cost` functions define a weighted graph without self-loops.
	//
	//  • edge(v, w) returns true whenever (v, w) belongs to the graph;
	//    the value is disregarded when v == w.
	//
	//  • cost(v, w) returns the cost of (v, w);
	//    the value is disregarded when edge(v, w) is false.
	//
	edge func(v, w int) bool
	cost func(v, w int) int64

	// The `degree` and `visit` functions can be used to improve performance.
	// They MUST BE CONSISTENT with edge and cost. If not implemented,
	// the `generic` or `generic0` implementation is used instead.
	// The `Consistent` test function should be used to check compliance.
	//
	//  • degree(v) returns the outdegree of vertex v.
	//
	//  • visit(v) visits all neighbors w of v for which w ≥ a in
	//    NUMERICAL ORDER calling do(w, c) for edge (v, w) of cost c.
	//    If a call to do returns true, visit MUST ABORT the iteration
	//    and return true; if successful it should return false.
	//    Precondition: a ≥ 0.
	//
	degree func(v int) int
	visit  func(v int, a int, do func(w int, c int64) (skip bool)) (aborted bool)
}
```

## Testing

## Optimization


#### Stefan Nilsson — [korthaj][korthaj]

*This work is licensed under a [Creative Commons Attribution 3.0 Unported License][ccby30].*

[ccby30]: https://creativecommons.org/licenses/by/3.0/
[ccbysa30]: https://creativecommons.org/licenses/by-sa/3.0/deed.en
[func]: https://github.com/yourbasic/func
[golang]: https://golang.org
[graph]: https://github.com/yourbasic/graph
[graphbuild]: https://github.com/yourbasic/graph/build
[graphdoc]: https://godoc.org/github.com/yourbasic/graph
[graphbuilddoc]: https://godoc.org/github.com/yourbasic/graph/build
[korthaj]: https://github.com/korthaj
[wikipetersen]: https://en.wikipedia.org/wiki/File:Petersen1_tiny.svg

