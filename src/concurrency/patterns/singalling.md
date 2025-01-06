# Signalling

Sometimes just closing a channel is enough to signal an event.

## 1. Wait For An Event

```go

//...

c := make(chan struct{})

go func() {
    //... do some stuff
    close(c)
}()

//... wait for goroutine  to finish

<-c

```

## 2. Coordinate Multiple Goroutines

```go

func worker(start chan bool) {
    <-start

    //... do stuff
}

func main() {
    start := make(chan bool)

    for i := 0; i < 1000; i++ {
        go worker(start)
    }

    //... workers are waiting for signal to start
    
    close(start)

    //...all workers are running now

    //...
}

```
