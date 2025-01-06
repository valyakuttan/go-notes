# Method Receivers

You can declare methods with pointer receivers or value receivers.

Methods with pointer receivers can modify the value to which the
receiver points. Since methods often need to modify their receiver,
pointer receivers are more common than value receivers.

The rule about pointers vs. values for receivers is that value methods can
be invoked on pointers and values, but pointer methods can only be
invoked on pointers.

This rule arises because pointer methods can modify the receiver; invoking
them on a value would cause the method to receive a copy of the value, so
any modifications would be discarded. The language therefore disallows this
mistake. There is a handy exception, though. When the value is **addressable**,
the language takes care of the common case of invoking a pointer method on
a value by inserting the address operator automatically.

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
