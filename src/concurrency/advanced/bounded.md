# Bounded Concurrency

A concurrent implementation with unbounded number of goroutines may hit
by resource limits, like allocating more memory than available on a machine.

```go
// ...

func run(in <-chan int) <-chan int {
    for data := range in {
        go func() {
            out <- process(data)
        }()
    }
}

// ...
```

We can limit the resource use by creating a fixed number of goroutines.

```go
// ...

func run(done <-chan struct{}, in <-chan int) <-chan int {
    numWorkers := 4
    out := make(chan int)
    var wg sync.WaitGroup
    wg.Add(numWorkers)
    for range numWorkers {
        go worker(done, in, out, &wg)
    }

    go func() {
        wg.Wait()
        close(out)
    }()

    return out
}

func worker(done <-chan struct{}, in <-chan int, out chan<- int, wg *sync.WaitGroup) {
    defer wg.Done()

    for {
        select {
        case data := <-in:
            out <- process(data)
        case <-done:
            return
        }
    }
}

// ...
```
