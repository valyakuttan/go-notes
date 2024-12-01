# Generics

## Type parameters

Go functions can be written to work on multiple types using type
parameters. The type
parameters of a function appear between brackets, before the
function's arguments.

```go

func Index[T comparable](s []T, x T) int {
    //...
}

```

This declaration means that `s` is a slice of any type `T` that fulfills
the built-in
constraint comparable. `x` is also a value of the same type.

comparable is a useful constraint that makes it possible to use
the `==` and `!=` operators
on values of the type. In this example, we use it to compare a
value to all slice
elements until a match is found. This Index function works for any
type that supports comparison.

## Generic types

In addition to generic functions, `Go` also supports generic types.
A type
can be parameterized with a type parameter, which could be useful
for implementing generic data structures.

```go

type Node[T comparable] struct {
 next *List[T]
 elem T
}

type List[T comparable] struct {
 head *Node[T]
}

func NewList[T comparable]() *List[T] {
 return new(List[T])
}

func NewNode[T comparable](val T) *Node[T] {
 n := new(Node[T])
 n.elem = val
 return n
}

func (lst *List[T]) Push(elem T) {
 l := NewList[T]()
 l.head = lst.head

 n := NewNode(elem)
 n.next = l
 lst.head = n
}

func (lst *List[T]) Pop() (T, error) {
 if lst.head == nil {
  var t T
  return t, fmt.Errorf("can't pop from an empty list")
 }

 n := lst.head
 lst.head = n.next.head

 return n.elem, nil
}

func (lst *List[T]) Print() {
 if lst.head != nil {
  fmt.Print(lst.head.elem, " -> ")
  lst.head.next.Print()
 } else {
  fmt.Println(nil)
 }
}

func main() {
 lst := NewList[int]()

 lst.Push(1)
 lst.Push(2)
 lst.Push(3)

 lst.Print()
 x, _ := lst.Pop()
 lst.Print()
 fmt.Println(x)
}

```
