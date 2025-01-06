
# The `Subscribe` Fucntion

`Subscribe` transforms a `Fetcher` into a `Subscription`. `Subscribe`
construct an unexported data type `naiveSub`, which implements
`Subscription`, and starts a goroutine `loop` which takes care of
handling all events.

To implement the `Subscription` interface, `Updates` and `Close` must
be defined.

## 1. The `Updates` method

```go

func (s *naiveSub) Updates() <-chan Item {
    return s.updates
}

```

## 2. The `Close` method

```go

func (s *naiveSub) Close() error {
    s.closed = true
    return s.err
}

```

## 3. The `loop` method

`loop` will

- periodically call `Fetch`
- send fetched items on the `Updates` channel
- exit when `Close` is called, reporting any error

```go

func (s *naiveSub) loop() {
    for {
        if s.closed {
            close(s.updates)
            return
        }
        items, next, err := s.fetcher.Fetch()
        if err != nil {
            s.err = err
            time.Sleep(10 * time.Second)
            continue
        }
        for _, item := range items {
            s.updates <- item
        }
        if now := time.Now(); next.After(now) {
            time.Sleep(next.Sub(now))
        }
    }
}

```
