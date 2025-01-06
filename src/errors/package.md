# Package errors

Package errors implements functions to manipulate errors.

## New

The New function creates errors whose only content is a text message.

## Wrapping erros

An error `e` wraps another error if `e`'s type has one of the methods

```go

Unwrap() error
Unwrap() []error

```

If `e.Unwrap()` returns a non-nil error `w` or a slice containing `w`, then
we say that `e` wraps `w`. A `nil` error returned from `e.Unwrap()` indicates
that `e` does not wrap any error. It is invalid for an Unwrap method to
return an `[]error` containing a `nil` error value.

An easy way to create wrapped errors is to call `fmt.Errorf` and apply the
`%w` verb to the error argument:

```go

wrapsErr := fmt.Errorf("... %w ...", ..., err, ...)

```

Successive unwrapping of an error creates a tree. The `Is` and `As` functions
inspect an error's tree by examining first the error itself followed by the
tree of each of its children in turn (pre-order, depth-first traversal).

## Is

`Is` examines the tree of its first argument looking for an error that matches
the second. It reports whether it finds a match. It should be used in
preference to simple equality checks:

```go

if errors.Is(err, fs.ErrExist)

```

## As

`As` examines the tree of its first argument looking for an error that can be
assigned to its second argument, which must be a pointer. If it succeeds,
it performs the assignment and returns `true`. Otherwise, it returns `false`.

```go

var perr *fs.PathError

if errors.As(err, &perr) {
    fmt.Println(perr.Path)
}

```
