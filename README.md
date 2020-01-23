## learnGo

- [Background](#background)
  * [Overview](#open-source)
  * [Open Source](#open-source)
  * [Standard Library](#standard-library)
- [Go Tour](#go-tour)
- [Go Playground](#go-playground)
- [Slices](#slices)
- [Maps](#maps)
- [Struct Tags](#struct-tags)
- [Packaging](#packaging)
  * [Importing](#importing)
  * [Exporting](#exporting)
  * [Design](#design)
  * [Structure](#structure)
- [Method Receivers](#method-receivers)
- [Variable Shadowing](#variable-shadowing)
- [Globals](#globals)
- [Generics](#generics)
- [Testing](#testing)
- [Common Mistakes](#common-mistakes)
- [Style Guide](#style-guide)

## Background

### Overview

Go is an open source programming language that makes it easy to build simple, reliable, and efficient software.
Go is a statically-typed, compiled language.
Go simplifies concurrent programming through [goroutines](https://tour.golang.org/concurrency/1) managed by the Go runtime.
The Go runtime is bundled in all Go binaries, which makes the starting size ~2Mb.

### Open Source

Go is an [open source programming language](https://github.com/golang/go/).

### Standard Library

The Go [standard library](https://golang.org/pkg/) is great.
The following are just a few of the libraries provided:
- [bytes](https://golang.org/pkg/bytes/) - Manipulation of bytes
- [context](https://golang.org/pkg/context/) - Used to carry deadlines/signals between processes
- [encoding](https://golang.org/pkg/encoding/) - Encoding types (e.g. `JSON`, `XML`)
- [crypto](https://golang.org/pkg/crypto/) - Hashing and random numbers
- [flag](https://golang.org/pkg/flag/) - Simplifies command-line flag parsing
- [io](https://golang.org/pkg/io/) - I/O primitives
- [fmt](https://golang.org/pkg/fmt/) - Standard I/O functions
- [math](https://golang.org/pkg/math/) - Math functions
- [net](https://golang.org/pkg/net/) - HTTP client/server implementations and related functions
- [os](https://golang.org/pkg/os/) - Platform independent OS function
- [reflect](https://golang.org/pkg/reflect/) - Runtime reflection
- [regexp](https://golang.org/pkg/regexp/) - Regex
- [runtime](https://golang.org/pkg/runtime/) - Access to the Go runtime
- [sort](https://golang.org/pkg/sort/) - Sorting for primitives
- [strconv](https://golang.org/pkg/strconv/) - Conversions to/from `string`
- [strings](https://golang.org/pkg/strings/) - String manipulation
- [sync](https://golang.org/pkg/sync/) - Sychronization primitives and atomic operations
- [testing](https://golang.org/pkg/testing/) - Unit testing and benchmarking
- [time](https://golang.org/pkg/time/) - Time


## Go Tour

Check out the [Go Tour](https://tour.golang.org).
It provides an excellent introduction to Go.

## Go Playground

The [Go Playground](https://play.golang.org/) makes it easy to share code snippits or quickly test some functionality.
Note however, this environment uses the latest stable release of Go and time is also seeded.


## Slices

[Slices](https://golang.org/src/runtime/slice.go) are dynamically-sized **references** to a backing array.
A `nil` value is valid for reads, but not writes.

Slices can be created using the built-in [`make`](https://golang.org/pkg/builtin/#make) function or through the short hand declaration:
```go
    len := 0
    cap := 5
    s1 := make([]int, cap) // [0 0 0 0 0]
    s2 := make([]int, len, cap) // [|X X X X X], X: Unknown cell, |: End of slice view
    s3 := []int{0, 0, 0} // [0 0 0]
```

Slices can be updated by their index as well as through the built-in [`append`](https://golang.org/pkg/builtin/#append) function:
```go
    subslice := s1[1:] // [0 0 0 0]
    s1[1] = 1 // s1: [0 1 0 0 0], subslice: [1 0 0 0]
    s2 = append(s2, 0) // [0|X X X]
    s3 = append(s3, 0) // [0 0 0 0]
```

## Maps

[Maps](https://golang.org/src/runtime/map.go) are dynamically-sized **references** that associate keys to values _without order_.
A `nil` value is valid for reads, but not writes.

A key can be any [comparable type](https://golang.org/ref/spec#Comparison_operators), which omits:
- Slices
- Maps
- Functions
- Any type composed of the former types

Maps can be created using the built-in [`make`](https://golang.org/pkg/builtin/#make) function or through the short hand declaration:
```go
    cap := 1000 // Specify at least how large the initial map must be
    m1 := make(map[string]string, cap) // {| X X ...}, X: Unknown cell, |: End of map view
    m2 := map[string]string{
        "learn": "Go",
        // Other elements
    }
```

Maps are updated by their index and automatically resize to accomodate new indices:
```go
    m1["learn"] = "Go" // {"learn": "Go"}
    m2["write"] = "documentation" // {"learn": "Go", "write": "documentation"}
```

## Struct Tags

Struct tags are meta data associated with structs, which can be accessed at runtime using [reflect](https://golang.org/pkg/reflect/).
Tags are key-value pairs normally used to specify rules for encoding and decoding data.
Valid tags are outlined for each data format (e.g. [JSON](https://golang.org/pkg/encoding/json/#Marshal)).

## Packaging

### Importing

Import paths should refer to packages stored on the local file system.
To refer to external packages, they should be [vendored](https://golang.org/cmd/go/#hdr-Vendor_Directories).
[Circular dependencies](https://en.wikipedia.org/wiki/Circular_dependency) are not alllowed.

Packages are imported according to their [directory structure](https://golang.org/pkg/net/http/#pkg-subdirectories).
Packages can also be aliased (prevent packages collisions or shorten name) with `.` and `_` as special cases:
```go
import (
    "net/http"
    ht "net/http/httptest"
    "github.com/google/go-cmp/cmp"
    . "math" // Exported fields directly accessible (and potentially shadowed by current package) without a [package]. prefix
    _ "k8s.io/client-go/plugin/pkg/client/auth/gcp" // The package and any associated packages have init() side effects
)
```

### Exporting

To publically export/expose a name outside of a package, it must be capitalized (e.g. `fmt.Println`).
Further, any exported member should have [documentation](https://blog.golang.org/godoc-documenting-go-code).

### Design

Similar to interfaces, packages should be boundaries that well define their responsibilities.
Try to avoid creating mono-packages so that only the required functionality is imported.
Slim packages make easy to understand APIs that are hard to misuse.
However, packages should not be broken up if they cannot function/exist without each other.

### Structure

Checkout the standard [project-layout repository](https://github.com/golang-standards/project-layout).
This can vary from project to project, but generally most of it should be applicable.

The following are nearly standard structure:
- `/cmd` stores executables
- `/test` stores external tests (non-unit tests)
- `/pkg` the top or nearly top level directory for source code

## Method Receivers

A value receiver type will be shallow copied when passed into a method.
A pointer receiver will pass an address/reference/pointer type into a method.


Consider the following:
```go
type PipelineRun struct {
	metav1.TypeMeta
	metav1.ObjectMeta
	Spec PipelineRunSpec
	Status PipelineRunStatus
}

func (p PipelineRun) doSomething() {
    // Does something
}

func (p *PipelineRun) SetSpec(s PipelineRunSpec) {
    p.Spec = s
}
```

The `doSomething` method definition has a `PipelineRun` **value receiver**, which means the `PipelineRun` will be copied.
The `PipelineRun` struct has two fields (`spec` and `status`) and two embedded types (`metav1.TypeMeta` and `metav1.ObjectMeta`), which makes this a relatively expensive operation.
Method receivers are passed alongside any parameters, which themselves follow these same copying rules.
For example, the `SetSpec` method definition (pointer receiver) has a `PipelineRunSpec` parameter that is passed by value (copied).

Using a value receiver can be a good fit when:
- The method is not modifying state
- The value is relatively small (avoids pointer indirection)

Use the same receiver type for all methods if possible.

## Variable Shadowing

Variable names `shadow` values of the same name within parent scope(s).
This makes the previous values inaccessible:
```go
package main

import (
	"fmt"
)

var x int

func main() {
    fmt.Println(x) // 0
	x := 1
    fmt.Println(x) // 1
	{
	    x := 2
        fmt.Println(x) // 2
	}
    fmt.Println(x) // 1
    fmt := "string"
    fmt.Println(fmt) //fmt.Println undefined (type string has no field or method Println)
}
```

## Globals

There are a lot of [opinions on global state](https://google.com/search?q=is+global+state+bad).
Consider alternatives if possible when creating globally mutable variables, _especially_ for exported values.

## Generics

Go does not have [generics](https://blog.golang.org/why-generics), but there is [a draft proposal](https://go.googlesource.com/proposal/+/master/design/go2draft-contracts.md).
To achieve the same effect, it is a common practice to use code generators such as [kubernetes/code-generator](https://github.com/kubernetes/code-generator) or [gen](https://github.com/clipperhouse/gen).

## Testing

Go has great testing built into the standard library.
The [`testing`](https://golang.org/pkg/testing/) package provides support for unit testing and benchmarking.

It is idiomatic to write [table driven tests](https://github.com/golang/go/wiki/TableDrivenTests).
When comparing for equality, favor [github.com/google/go-cmp/cmp](https://github.com/google/go-cmp/tree/master/cmp) over [reflect](https://golang.org/pkg/reflect/).

## Common Mistakes

Check out these [common mistakes](https://github.com/golang/go/wiki/CommonMistakes).

## Style Guide

- Verbose: [Effective Go](https://golang.org/doc/effective_go.html)
- TLDR: [Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)