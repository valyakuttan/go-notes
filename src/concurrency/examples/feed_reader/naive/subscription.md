
# The `Subscription` Interface

`Subscription` interface provides a continuous stream of updates and an 
ability to stop receiving updates.

```go

type Subscription interface {
    Updates() <-chan Item // stream of Items
    Close() error         // shuts down the stream
}

func Subscribe(fetcher Fetcher) Subscription {...} // converts Fetches to a stream

func Merge(subs ...Subscription) Subscription {...} // merges several streams


```
