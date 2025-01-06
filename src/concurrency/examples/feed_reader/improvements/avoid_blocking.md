# Avoid Blocking On Fetch

## Issue: Fetch May Block the Goroutine

Fetches needs network operations which may take long time. So
the call to `Fetch` may block, which may affect the responsiveness
of the system.

```go

case <-startFetch:
    //...
    fetched, next, err = s.fetcher.Fetch()
    if err != nil {
    //..
    }

    
    for _, item := range fetched {
        if !seen[item.GUID] {
           //...     
        }
    }

```

## Fix: Run Fetch Asynchronously

We move `Fetch` to its own goroutine. Now we need to know when the fetch is
finished to resynchronize with the working of `loop`. This is done by another
case into `select` and another channel to communicate with the fetching
goroutine.

```go

type fetchResult struct{ fetched []Item; next time.Time; err error }

var fetchDone chan fetchResult // if non-nil, Fetch is running

//...

    var startFetch <-chan time.Time
        if fetchDone == nil && len(pending) < maxPending {
            startFetch = time.After(fetchDelay) // enable fetch case
        }

        select {
        
        //......
        
        case <-startFetch:
            fetchDone = make(chan fetchResult, 1)
            go func() {
                fetched, next, err := s.fetcher.Fetch()
                fetchDone <- fetchResult{fetched, next, err}
            }()
        
        case result := <-fetchDone:
            fetchDone = nil
            // Use result.fetched, result.next, result.err

        //....

        }
    
    //...


```
