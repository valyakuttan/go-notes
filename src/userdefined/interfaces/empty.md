# The empty interface

The interface type that specifies zero methods is known as the empty
interface:

```go
interface{} // same as the type any
```

An empty interface may hold values of any type.
Empty interfaces are used by code that handles values of unknown type.

```go

var i1 interface{} // nil interface value

fmt.Println("a == nil: ", i1 == nil) 
// a == nil:  true

var i2 interface{}
var p *int = nil
i2 = p // interface holding nil

fmt.Println("b == nil: ", i2 == nil)
// b == nil:  false

```
