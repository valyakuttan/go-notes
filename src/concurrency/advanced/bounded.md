# Bounded Concurrency using Semaphore

A counting semaphore helps to restricts the number of concurrently executing
threads. Buffered channels make it easy to create counting semaphore.

```go

// ...

// We want information about all cities, a single failure
// make the result unusable
func Cities(cities ...string) ([]*Info, error) {
    var g errgroup.Group
    var mu sync.Mutex
    res := make([]*Info, len(cities)) // res[i] corresponds to cities[i]
    sem := make(chan struct{}, 10)
    for i, city := range cities {
        i, city := i, city // create locals for closure below
        sem <- struct{}{}
        g.Go(func() error {
            info, err := City(city)
            mu.Lock()
            res[i] = info
            mu.Unlock()
            <-sem
            return err
        })
    }
    if err := g.Wait(); err != nil {
        return nil, err
    }
    return res, nil
}


// ...

```
