# Your basic func [sketch]

### Building graphs from functions only

[![Cartesian product](res/petersen2.png)](#table-of-contents)

**The Petersen graph and its complement**,
*picture from [Wikipedia][wikipetersen2], [CC BY-SA 3.0][ccbysa30].*


## Table of contents

- [x] [Introduction](#introduction)
- [ ] [Implementation](#implementation)
- [ ] [Testing](#testing)
- [ ] [Performance](#performance)


## Introduction

This text is about the implementation of a Go tool based entirely on functions –
the API contains only immutable data types, and the code
is built on top of a `struct` with five `func` fields.

It's a tool for building **virtual graphs**.
In a virtual graph no vertices or edges are stored in memory,
they are instead computed as needed.
The tool is part of a larger library of generic graph algorithms:

- the package [graph][graph] contains the basic graph library and
- the subpackage [graph/build][graphbuild] is the tool for building virtual graphs.

There is an online reference for the build tool at [godoc.org][graphbuilddoc].


### Petersen graph

Let's start with an example, the Petersen graph.
To describe this graph in a conventional graph library,
you would typically need to enumerate the edges

	{0, 1}, {0, 4}, {0, 5}, {1, 2}, {1, 6}, {2, 3}, {2, 7}, {3, 4},
	{3, 8}, {4, 9}, {5, 7}, {5, 8}, {6, 8}, {6, 9}, and {7, 9}

in one way or another.

However, if you ask a mathematician to describe the Petersen graph,
her answer would clearly be very different. Perhaps she would say:

> You get a Petersen graph if you draw a pentagon
> with a pentagram inside, with five spokes.

![Petersen graph](res/petersen.png)

*Picture from [Wikipedia][wikipetersen], [CC BY-SA 3.0][ccbysa30].*

This [example][petersendoc] from the `graph/build` documentation
is very close to this mathematical description.

```go
// Build a Petersen graph.
pentagon := build.Cycle(5)
pentagram := pentagon.Complement()
petersen := pentagon.Match(pentagram, build.AllEdges())
```

In fact, the functions implement basic concepts in graph theory:
we start with a **cycle graph** of order 5,
compute its **complement**, and then combine these two graphs
by **matching** their vertices.

### Generic graphs

It's also possible to define new graphs by
writing **functions** that describe their **edge sets**.
The code in the following [example][genericdoc]
builds a directed graph containing all edges (*v*, *w*)
for which *v* is odd and *w* even.

```go
// Define a graph by a function.
g := build.Generic(10, func(v, w int) bool {
    // Include all edges with v odd and w even.
    return v%2 == 1 && w%2 == 0
})
```

The `build.Generic` function returns a virtual graph with 10 vertices;
its edge set consists of all edges (v, w), v ≠ w, for 
which the anonymous function returns true.


## Implementation

The main data type is a struct with five functions.

Discussion...


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

### Cycle graphs

Let's look at a very simple example, how to implement a **cycle graph**,
a graph of order *n* with the edges  
{0, 1}, {1, 2}, {2, 3},... , {*n*-1, 0}.

For a minimal effort implementation of this class of graphs,
only the `edge` function needs to be defined –
`generic0` returns a standard implementation for the other functions.

```go
g := generic0(n, func(v, w int) (edge bool) {
	switch v - w {
	case 1 - n, -1, 1, n - 1:
		edge = true
	}
	return
})
```

In this case, it's trivial to compute the degree more efficiently.

```go
g.degree = func(v int) int { return 2 }
```

It's also easy to define a much more efficient `visit` function.
For example like this.

```go
g.visit = func(v int, a int, do func(w int, c int64) bool) (aborted bool) {
	var w [2]int
	switch v {
	case 0:
		w = [2]int{1, n - 1}
	case n - 1:
		w = [2]int{0, n - 2}
	default:
		w = [2]int{v - 1, v + 1}
	}
	for _, w := range w {
		if w >= a && do(w, 0) {
			return true
		}
	}
	return
}
```


## Testing

As usual, we need to check that the function calls
`Cycle(n)` produce the expected result.
However, checking for consistency can be fully automated:

```go
for n := 0; n < 5; n++ {
	Consistent("Cycle", t, Cycle(n))
}
```

The `Consistent` function checks that the `degree` and `visit` functions
are consistent with `edge` and `cost`, and that `visit` obeys its contract.

This is nice, since the two definitions tend to be quite different and
often one of the two is considerably easier to implement.


## Performance

Caching. Hard to automate. As always, it's hard to know what to cache.
In this case, any graph, or part of graph, which is described by `edge`
functions cannot be cached, since we can't assume to that user-provided
functions are pure – the may return different results given the same argument.

The solution: the `Specific` function.

```go
// Specific returns a cached copy of g with constant time performance for
// all basic operations. It uses space proportional to the size of the graph.
func Specific(g graph.Iterator) *Virtual
```

It can be used to cache any component of a virtual graph.
It also solves a second problem, importing graphs that are represented
by more traditional data structures.


#### Stefan Nilsson — [korthaj][korthaj]

*This work is licensed under a [Creative Commons Attribution 3.0 Unported License][ccby30].*

[ccby30]: https://creativecommons.org/licenses/by/3.0/
[ccbysa30]: https://creativecommons.org/licenses/by-sa/3.0/deed.en
[func]: https://github.com/yourbasic/func
[genericdoc]: https://godoc.org/github.com/yourbasic/graph/build#example-Generic
[golang]: https://golang.org
[graph]: https://github.com/yourbasic/graph
[graphbuild]: https://github.com/yourbasic/graph/tree/master/build
[graphdoc]: https://godoc.org/github.com/yourbasic/graph
[graphbuilddoc]: https://godoc.org/github.com/yourbasic/graph/build
[korthaj]: https://github.com/korthaj
[petersendoc]: https://godoc.org/github.com/yourbasic/graph/build#example-Virtual-Match-Petersen
[wikipetersen]: https://en.wikipedia.org/wiki/File:Petersen1_tiny.svg
[wikipetersen2]: https://commons.wikimedia.org/wiki/File:Petersen_graph_complement.svg

