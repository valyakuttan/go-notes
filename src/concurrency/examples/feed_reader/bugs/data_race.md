# 1. Data races

`s.closed` and `s.err` are accessed by two goroutines without
synchronization, which leads to **data race**.

```go

func (s *naiveSub) loop() {
    for {
        //....
       if s.closed {

          //...
       }

       //...
       
       if err != nil {
           s.err = err
            
            //...
       }

       //...

    }
}

//...

func (s *naiveSub) Close() error {
    s.closed = true
    return s.err
}

```

We can use **Go**`s race detector to find data races, which
can be invoked by

```bash

go run -race app.go

```
