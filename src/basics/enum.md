# Enumerated Constants

In **Go**, enumerated constants are created using the `iota` enumerator.
Since `iota` can be part of an expression and expressions can be
implicitly repeated, it is easy to build intricate sets of values.

```go

type ByteSize float64

const (
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)

```
