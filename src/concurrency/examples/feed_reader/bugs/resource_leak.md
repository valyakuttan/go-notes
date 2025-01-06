# 2. Resource leak

When there is nothing to do, the goroutine which runs the `loop` will go
to sleep and will ignore all requests, which is problematic when a
close request comes. Closing happen only during next fetch, which
may occur after an arbitrary long duration. This may leads to resource
leak.

```go

func (s *naiveSub) loop() {
    for {

        //...

        if err != nil {
           
           //...
           time.Sleep(10 * time.Second)
           continue
        }

        //...

        if now := time.Now(); next.After(now) {
            time.Sleep(next.Sub(now))
        }
    }
}

//...

```
