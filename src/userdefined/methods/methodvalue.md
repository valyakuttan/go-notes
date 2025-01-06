# Method Values

Method values allow you to bind a method to a specific object, and then
call the method as an ordinary function with the object implied, in a
kind of closure.

```go

type Rocket struct { /* ... */ }

func (r *Rocket) Launch() {}

r := new(Rocket)

// we launch rocket after 10 seconds countdown
time.AfterFunc(10*time.Second, func() {r.Launch()})

// the above code can be written much nicer way with the help of
// a method value
time.AfterFunc(10*time.Second, r.Launch)

```
