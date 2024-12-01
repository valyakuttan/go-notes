# Constants

In **Go**, `const` is a keyword introducing a name for a scalar value
like `1`, `3.1415` or a string like `"hello"`. They are created at
compile time, even when defined as locals in functions, and can only
be numbers, characters (runes), strings or booleans.

The expression

```go
const hello = "Hello, World!"

const typedHello string = "Hello, World!"

```

creates an untyped string constant `hello` and a typed string constant
`typedHello`.

```go
type MyString string

var m MyString
m = typedHello // Type error

m = MyString(typedHello) // OK

m = hello // ok, hello is untyped

```
