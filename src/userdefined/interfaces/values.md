# Interface Values

Under the hood, interface values can be thought of as a tuple of a value
and a concrete type:

`(value, type)`

An interface value holds a value of a specific underlying concrete type.

Calling a method on an interface value executes the method of the same name
on its underlying type.

## Interface values with `nil` underlying values

If the concrete value inside the interface itself is `nil`, the method will
be called with a `nil` receiver.

In some languages this would trigger a null pointer exception, but in Go it
is common to write methods
that gracefully handle being called with a `nil` receiver.

Note that an interface value that holds a `nil` concrete value is itself
non-nil.

## Nil interface values

A nil interface value holds neither value nor concrete type.

Calling a method on a nil interface is a run-time error because there is
no type inside the interface
tuple to indicate which concrete method to call.
