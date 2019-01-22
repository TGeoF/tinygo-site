---
title: "Heap allocation"
weight: 2
---

Many operations in Go rely on heap allocation. Some of these heap allocations are optimized away, but not all of them. Also, TinyGo does not yet contain a garbage collector so heap allocation must be avoided whenever possible outside of initialization code.

These operations currently do heap allocations:

* Taking the pointer of a local variable. This will result in a heap allocation, unless the compiler can see the resulting pointer never escapes. This causes a heap allocation:

```go
var global *int

func foo() {
    i := 3
    global = &i
}
```

This does not cause a heap allocation:

```go
func foo() {
    i := 3
    bar(&i)
}

func bar(i *int) {
    println(*i)
}
```

* Converting between `string` and `[]byte`. In general, this causes a heap allocation because one is constant while the other is not: for example, a `[]byte` is not allowed to write to the underlying buffer of a `string`. However, there is an optimization that avoids a heap allocation when converting a string to a `[]byte` when the compiler can see the slice is never written to. For example, this ``WriteString`` function does not cause a heap allocation:

```go
func WriteString(s string) {
    Write([]byte(s))
}

func Write(buf []byte) {
    for _, c := range buf {
        WriteByte(c)
    }
}
```

* Converting a `byte` or `rune` into a `string`. This operation is actually a conversion from a Unicode code point into a single-character string so is similar to the previous point.

* Concatenating strings, unless one of them is zero length.

* Creating an interface with a value larger than a pointer. Interfaces in Go are not a zero-cost abstraction and should be used carefully on microcontrollers.

* Closures where the collection of shared variables between the closure and the main function is larger than a pointer.

* Creating and modifying maps. Maps have *very* little support at the moment and should not yet be used. They exist mostly for compatibility with some standard library packages.

* Starting goroutines. There is limited support for goroutines and currently they are not at all efficient. Also, there is no support for channels yet so their usefulness is limited.