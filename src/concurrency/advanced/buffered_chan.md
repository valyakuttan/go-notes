# Buffered Channel Trick

Buffered channel with capacity `1` can be used to dispose a goroutine
as soon as send is done.

```go

//...
ch := make(chan int, 1)

go func() {
    ch <- 1 // will not block for a receive
}()

```
