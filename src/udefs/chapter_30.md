# Methods

Go does not have classes. However, you can define methods on types.

A method is a function with a special receiver argument.

The receiver appears in its own argument list between the func
keyword and the method name.

In this example, the Abs method has a receiver of type Vertex named v.

Note: A method is just a function with a receiver argument.

You can only declare a method with a receiver whose type is defined in the
same package as the method. You cannot declare a method with a receiver
whose type is defined in another package.

## Pointer Receivers vs Value Receivers

You can declare methods with pointer receivers.

This means the receiver type has the literal syntax `*T` for some type `T`.

Methods with pointer receivers can modify the value to which the
receiver points. Since methods often need to modify their receiver,
pointer receivers are more common than value receivers.

Irrespective of the receiver type methods can call on both
value and pointer types.

In general, all methods on a given type should have either value
or pointer receivers, but not a mixture of both.

```go

type vertex1 struct {
 X, Y float64
}

func (v vertex1) abs() float64 {
 return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *vertex1) scale(f float64) {
 v.X *= f
 v.Y *= f
}

```
