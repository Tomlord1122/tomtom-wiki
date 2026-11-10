# Go Memory Model: Arrays, Slices, Strings, and Maps

Go's memory notes treat collections as different machines, not spellings of "list." An array is a value whose length is part of the type. A slice is a three-word header that looks at an array. A string is an immutable two-word view over bytes. A map is a handle to runtime-managed hash storage. `unsafe` is the escape hatch that makes those representations visible, and also the place where the garbage collector can no longer save you.

The Chapter 3 notes captured on 2026-08-28 add a reusable test: does the value own its elements or point at storage? What is copied when it is passed? Can operations share backing memory? What event reallocates, isolates mutation, or restructures behind the API?

Chapters 6–8 (functions, heap/GC, MPG scheduler) now live on [[go-runtime-anatomy]]. This page stays the collections machine.

## Arrays are values with a size in the type

`[5]int` and `[3]int` are different types. The compiler sees a contiguous block, not a pointer to a list. Assignment copies every element. Passing an array to a function copies the whole block unless the caller passes a pointer.

Initialization is a compiler choice, not a single runtime path. Small constant arrays can be materialized directly. Larger ones may be built in static data or copied onto the stack, then sometimes moved to the heap if the value escapes. Escape analysis, not the `new` keyword, decides stack versus heap. `len` and `cap` of a directly known array are compile-time facts.

This rigidity is the point. The size is known, the layout is obvious, and there is no hidden header. It is also why arrays are a poor API for variable-length data.

## The slice header

A slice value is always three machine words, 24 bytes on a 64-bit architecture:

- a data pointer to the first visible element
- `len`, the visible element count
- `cap`, how many elements exist from that pointer to the end of the backing array

Passing a slice copies the header, not the backing array. That is why a ten-million-element slice is cheap to pass and why two slices can share memory by accident. `b := a` for arrays copies elements. `b := a` for slices copies the window. For `arr[2:4]`, the pointer starts at `arr[2]`; length counts visible elements; capacity runs from that start to the end of available backing storage.

Slice expressions are compiler rewrites around that header. `s[i:j]` builds a new header whose pointer is `&s[i]` (or the equivalent address), whose length is `j-i`, and whose capacity is the remaining capacity from `i`. Full slice expressions (`s[i:j:k]`) cap how far a derived slice may grow. Reslicing can still see the old array.

`append` writes into unused capacity when it exists; only growth past `cap` allocates a new array and copies. Shared backing arrays plus `append` are the usual aliasing bug: aliases of the old storage keep pointing at the old array after a growth. A function may mutate elements in place, but if it appends it must return the new header or the caller never sees the growth. Capacity commonly doubles while small, then `runtime.nextslicecap` grows more conservatively after 256 elements and rounds to allocator size classes — so `[]byte`, `[]int32`, and `[]int64` do not show the same observed growth sequence. The practical limit is total bytes, not element count.

Inspecting the header with pointer arithmetic is educational only. Runtime layout is not a public contract.

## Strings are two-word immutable views

A string header stores a pointer and a byte length — 16 bytes on 64-bit, no capacity. Assignment and slicing copy the header, so two strings may share bytes. That is safe because ordinary Go cannot mutate string data; an apparent modification allocates a new string.

UTF-8 makes byte length, rune count, and visible characters different numbers. Conversions copy when they must, with compiler/runtime fast paths when size and lifetime allow. Concatenation folds compile-time constants early and uses stack buffers for short results.

Go will not let ordinary code take `&s[0]`. `unsafe.StringData` can expose the first byte, but writing through it is undefined and can crash, especially for literals in read-only memory.

## Maps are handles, not values

A map stores unordered key-value pairs. Keys must be comparable (`==` / `!=`); slices, maps, and functions cannot be keys. An `interface{}` key only postpones the check: a non-comparable dynamic value panics at hash time.

A nil map reads like empty (zero value, iteration, existence checks). Writing to a nil map panics. `make` or a map literal creates storage.

The visible map value points at an internal `hmap`: count, flags, power-of-two bucket count, overflow estimates, per-map hash seed, current and old bucket arrays, and gradual-evacuation progress. Each bucket holds up to eight pairs plus an eight-byte tophash array. Overflow buckets chain instead of resizing immediately. Keys/values larger than the described 128-byte threshold are stored by pointer so buckets stay scannable. Lookup compares the tophash fingerprint first, then the full key; reserved tophash values encode empty, moved, and evacuation states.

Insertion and deletion reuse bucket states while growth proceeds. Growth is gradual: add buckets or clean up at the same size, evacuating incrementally. Iteration is intentionally non-deterministic but remains coherent during ordinary safe use. The public API stays small while hiding substantial runtime work.

## Addressability

Not every Go value has an address. The rule is mechanical: the compiler must know there is a durable location.

Addressable: variables, pointer indirection, slice elements, addressable-struct fields, array elements of an addressable array.

Not addressable: map elements, function results, type conversions, and other temporaries. `&m[k]` is illegal because a map entry can move when the map grows. A method with a pointer receiver therefore cannot be called on a non-addressable value unless the compiler can materialize a temporary, which it will not do for every case.

This is why "it works on a variable but not on a map value or a function return" is not a style complaint. It is the memory model showing through the type system.

## `unsafe.Pointer` and `uintptr`

`unsafe.Pointer` is a pointer. The garbage collector still sees it and will not recycle the referent. `uintptr` is an integer that happens to hold an address. The GC does not treat it as a live reference.

The legal conversions are narrow: pointer to `unsafe.Pointer` to a different pointer type, or through `uintptr` only for immediate arithmetic that stays inside the same object. Storing a `uintptr` for later use, especially across a function call or after the original pointer died, is how memory is reused under you.

The notes pair this with integer overflow and IEEE-754 only as adjacent machine facts: wrapping and rounding are defined, not surprising, once you look at the bits. They do not change the slice-header story, but they explain why "the hardware did something weird" is usually "the representation did exactly what it says."

## Related pages

- [[go-runtime-anatomy]]: chapters 6–8 (functions, heap/GC, MPG) captured on go-backend.
- [[go-layered-backend]]: caller-owned interfaces sitting on this runtime.

## References

