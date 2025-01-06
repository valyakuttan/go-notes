# Errors

Go programs express error state with error values.

The error type is a built-in interface similar to `fmt.Stringer`:

```go

type error interface {
    Error() string
}

```

Functions often return an error value, and calling code should handle
errors by testing whether the error equals `nil`.

- A `nil` error denotes success; a non-nil error denotes failure.

- `fmt.Errorf` can be used to wrap an error, which returns an error type.

```go

_, error := some()
return fmt.Errorf("While performing %s, %w happened", "some", error)

```
