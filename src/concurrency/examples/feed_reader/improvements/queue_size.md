# Limit Pending Queue Size

## Issue: Pending Queue Size Grows Without Bounds

The set of pending items can grow without bounds. May be our downstream
receiver not keeping up with the updates. So we don't want to append
more pending items to the pending queue, which allocates more memory to
to hold items.

## Fix: Disable Fetch Case When Too Much Pending

```go

//...

const maxPending = 10

//...


   var fetchDelay time.Duration
    if now := time.Now(); next.After(now) {
        fetchDelay = next.Sub(now)
    }
    
    var startFetch <-chan time.Time
    if len(pending) < maxPending {
        startFetch = time.After(fetchDelay) // enable fetch case
    }

//....

```
