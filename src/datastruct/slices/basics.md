# Basics

An array has a fixed size. A slice, on the other hand, is a dynamically-sized,
flexible view into the elements of an array.

The type `[]T` is a slice with elements of type `T`.

# Slice Expressions

A slice is formed by specifying two indices, a low and high bound, separated
by a colon:

```go

 b := a[low : high]

```

This selects a half-open range which includes the first element, but excludes
the last one.

## Slice Length And Capacity

A slice has both a length and a capacity.

The length of a slice is the number of elements it contains.

The capacity of a slice is the number of elements in the underlying
array, counting from the first element in the slice.

The length and capacity of a slice `s` can be obtained using the
expressions `len(s)` and `cap(s)`.

The zero value of a slice is `nil`. The `len` and `cap` functions
will both return `0` for a `nil` slice.

## Dynamically sized arrays as slices

Slices can be created with the built-in `make` function.
The `make` function allocates a zeroed array and returns a slice that refers to that array:

```go

a := make([]int, 5)  // len(a)=5

```

To specify a capacity, pass a third argument to make:

```go

b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4

```

## Appending to a slice

It is common to append new elements to a slice, and so Go provides a built-in
`append` function.

```go

 var s []int // nil slice

 // append works on nil slices.
 s = append(s, 0)

 // The slice grows as needed.
 s = append(s, 1)

 // We can add more than one element at a time.
 s = append(s, 2, 3, 4)

```

If the backing array of s is too small to fit all the given values a bigger array
will be allocated. The returned slice will point to the newly allocated array.

# Controlling The Slice Capacity

The slice expression

```go

b := a[low: high : max]

```

will create slice of length `high - low` and having the capacity `max - low`,
which is an effective way to control the behavior of slice expansion during
`append`, which may cause surprises.

```go

a := [5]int{0, 1, 2, 3, 4}
b := a[:3:4]

fmt.Printf("b: %v, len(b): %d, cap(b): %d\n", b, len(b), cap(b))

// print
// b: [0 1 2], len(b): 3, cap(b): 4

```
