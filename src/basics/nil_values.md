# `nil` values

There is no `nil` type, nor can you alias another type to be `nil`, it is a
reserved word. `nil` can be assigned to a value, and values can be compared
to `nil`, but `nil` cannot be compared to itself.

- If you assign `nil` to a pointer the pointer will not point to anything.

- If you assign `nil` to an interface, the interface will not contain anything.

- If you assign `nil` to a slice and the slice will have zero `len` and
  zero `cap` and no longer point an underlying array.

## Don't Mix `nil` with `interfaces`

When you assign a `nil` to a variable of type interface it is no longer
a simple `nil` value. This may lead to surprising outcomes.

Since an interface has  a type and a value if we assign a value with
a concrete type to an interface variable it is no longer a `nil` value,
it has a concrete type, so it is different.

If your method returns an interface type, be sure to always return `nil`
explicitly.

```go
func Open(path string) io.Writer {
    var f *os.File
    f, _ = os.Open(path)
    return f
}

func main() {
    f := Open("/missing")
    fmt.Println(f == nil) // print false
}
```
