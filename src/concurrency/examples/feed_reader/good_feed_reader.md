# A Good Feed Rader

The bugs can be fixed by changing the body of `loop` to a `select`
with three cases:

- `Close` was called
- it's time to call `Fetch`
- send an item on `s.updates`

## Structure: for-select loop

loop runs in its own goroutine.

select lets loop avoid blocking indefinitely in any one state.

```go

func (s *sub) loop() {
    //... declare mutable state ...
    for {
        .//.. set up channels for cases ...
        select {
        case <-c1:
            //... read/write state ...
        case c2 <- x:
            //... read/write state ...
        case y := <-c3:
            //... read/write state ...
        }
    }
}

```

The cases interact via local state in loop. So there is no data races.
