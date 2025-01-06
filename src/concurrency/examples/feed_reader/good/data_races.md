# 1. Fixing The Data Races

We replaced the manipulation of shared variables `s.closed` and `s.err`
with communication along a channel.

`Close` communicates with `loop` via `s.closing`.

```go

type sub struct {
    closing chan chan error
}

```

This is a **Request-Response** structure. The `loop` is a server which
listens for requests on its channel `s.closing`.The client
(`Close`) sends a request on `s.closing`. When  the `loop` acknowledges the
request by sending an `error` value `Close` can safely exit by returning the
`error` value obtained.

In this case, only thing in the request is the reply channel.

## 1.1 `Close` asks loop to exit and waits for a response
 
`Close` creates a channel and send it to the `loop` and waits for an
`error` value on it. Once obtained return the value to the caller.

```go

func (s *sub) Close() error {
    errc := make(chan error)
    s.closing <- errc
    return <-errc
}

```

## 1.2 `loop` handles `Close` by replying with the `Fetch` error and exiting

```go

    //...

    var err error // set when Fetch fails
    for {
        select {
        case errc := <-s.closing:
            errc <- err
            close(s.updates) // tells receiver we're done
            return
        
        //...

        }
    }

```
