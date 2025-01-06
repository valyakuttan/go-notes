# Deduplicate Items

## Issue: Fetch May Return Duplicates

We don't want to stream duplicates on the update channel. We can filter
items before adding to the pending items.

## Fix: Filter Items Before Adding To Pending Queue

```go

//....

var pending []Item
var next time.Time
var err error
var seen = make(map[string]bool) // set of item.GUIDs

//....

case <-startFetch:
    var fetched []Item
    fetched, next, err = s.fetcher.Fetch()
    if err != nil {
        next = time.Now().Add(10 * time.Second)
        break
    }

    for _, item := range fetched {
        if !seen[item.GUID] {
            pending = append(pending, item)
            seen[item.GUID] = true
        }
    }

//...

```
