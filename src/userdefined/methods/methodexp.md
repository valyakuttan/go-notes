# Method Expressions

Method expressions let you transform a method into a function with the
receiver as its first argument. But if the method has a pointer receiver,
you need to write (*Type).Method in your method expression.

```go

type Rocket struct { /* ... */ }

func (r *Rocket) Launch() {}

r := new(Rocket)

f := Rocket.Launch

// we launch rocket after 10 seconds countdown
time.AfterFunc(10*time.Second, func() {f(r)})

```
