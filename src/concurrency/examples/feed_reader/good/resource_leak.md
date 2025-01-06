# 2. Fixing The Resource Leak

Resource Leak is fixed by modifying the fetch case.

The mutable state consists of

1. `pending` - A set of items which is to be delivered to the client, which
    initially empty.

2. `next` - Time at which we perform our next fetch

3. `err` - The failure status of previous fetch operation

`startFetch` is a channel which receives a value when `fetchDelay` elapsed.

```go

func (s *sub) loop() {
    var pending []Item
    var next time.Time
    var err error
    for {
        var fetchDelay time.Duration
        if now := time.Now(); next.After(now) {
            fetchDelay = next.Sub(now)
        }
        startFetch := time.After(fetchDelay) // enable fetch case

        //...

        select {
        case <-startFetch:
            var fetched []Item
            fetched, next, err = s.fetcher.Fetch()
            if err != nil {
                next = time.Now().Add(10 * time.Second)
            }
            pending = append(pending, fetched...)
        
        //...

        }
    }
}


```
