# User Defined Types

Go supports user defined types in the form of a `struct` which can
have zero or more fields.

```go
type Vertex struct {
 X int
 Y int
}
```

## Embedding

Including an anonymous field in a `struct` is known as **embedding**.
The main purpose of type embedding is to extend the functionalities of
the embedded types into the embedding type, so that we don't need
to re-implement the functionalities of the embedded types for the
embedding type.

```go

type Foo struct {
 f string
}

func(f Foo) foo() string {
 return f.f
}
type Bar struct {
 b string
}

func(b Bar) bar() string {
 return b.b
}

type Baz struct {
 Foo
 *Bar
}

 baz := Baz{Foo{"foo"},&Bar{"bar"}}

 baz.foo() // same as baz.Foo.foo()
 // Note: anonymous fields can be referenced
 // with their type name like baz.Foo

 baz.bar() // same as baz.Bar.bar()
 
```
