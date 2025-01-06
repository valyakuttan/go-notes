# Cancellation, Context, And Plumbing

In Go servers, each incoming request is handled in its own goroutine. Request
handlers often start additional goroutines to access backends such as
databases and RPC services. The set of goroutines working on a request
typically needs access to request-specific values such as the identity of
the end user, authorization tokens, and the request’s deadline. When a
request is canceled or times out, all the goroutines working on that request
should exit quickly so the system can reclaim any resources they are using.

The context package makes it easy to pass request-scoped values,
cancellation signals, and deadlines across API boundaries to all the
goroutines involved in handling a request.

A Context carries a cancellation signal and request-scoped values to all
functions running on behalf of the same task. It's safe for concurrent
access.

```go

// A Context carries a deadline, cancellation signal, and request-scoped values
// across API boundaries. Its methods are safe for simultaneous use by multiple
// goroutines.
type Context interface {
    // Done returns a channel that is closed when this Context is canceled
    // or times out.
    Done() <-chan struct{}

    // Err indicates why this context was canceled, after the Done channel
    // is closed.
    Err() error

    // Deadline returns the time when this Context will be canceled, if any.
    Deadline() (deadline time.Time, ok bool)

    // Value returns the value associated with key or nil if none.
    Value(key interface{}) interface{}
}

```

## Context Usage Notes

- Idiom: pass ctx as the first argument to a function.

- Context has no Cancel method; obtain a cancelable Context using `WithCance`

## Example

For the search example given below, even after the `first` returns, the remaining searches
may continue running, which uses resources unnecessarily.

```go

// Search runs query on a backend and returns the result.
type Search func(query string) Result
type Result struct {
    Hit string
    Err error
}

// First runs query on replicas and returns the first result.
func First(query string, replicas ...Search) Result {
    c := make(chan Result, len(replicas))
    search := func(replica Search) { c <- replica(query) }
    for _, replica := range replicas {
        go search(replica)
    }
    return <-c
}

```

By adding a cancellable context we can stop the unnecessary replica searches.

```go

// Search runs query on a backend and returns the result.
type Search func(ctx context.Context, query string) Result

// First runs query on replicas and returns the first result.
func First(ctx context.Context, query string, replicas ...Search) Result {
    c := make(chan Result, len(replicas))
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()
    search := func(replica Search) { c <- replica(ctx, query) }
    for _, replica := range replicas {
        go search(replica)
    }
    select {
    case <-ctx.Done():
        return Result{Err: ctx.Err()}
    case r := <-c:
        return r
    }
}

```
