# Arrays

The type `[n]T` is an array of `n` values of type `T`.

```go

var a [10]int

```

An array's length is part of its type, so arrays cannot be resized.

Arrays do not need to be initialized explicitly; the zero value of
an array is a ready-to-use array whose elements are themselves zeroed.

**Go**’s arrays are values. An array variable denotes the entire array;
it is not a pointer to the first array element (as would be the case
in C). This means that when you assign or pass around an array value
you will make a copy of its contents.

An array literal can be specified like so:

```go

b := [2]string{"Penn", "Teller"}

```

Or, you can have the compiler count the array elements for you:

```go

b := [...]string{"Penn", "Teller"}

```

In both cases, the type of `b` is `[2]string`.
