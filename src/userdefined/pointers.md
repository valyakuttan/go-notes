# Pointers

Pointers allow operating on data structures without the need
of expensive copying.

```go

type Vertex struct {
 X int
 Y int
}

func main() {
 var v *Vertex = new(Vertex) // or v := new(Vertex)
 // v.X, v.Y are 0, 0

}

```
