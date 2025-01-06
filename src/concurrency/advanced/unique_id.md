# Unique ID Service

- Receive unique id from a channel

- Safe to share across gorountines

```go

//...

id := make(chan string)

//...

go func() {
    var counter int64
    for {
        id <- fmt.Sprintf("%x", counter)
        counter += 1
    }
}()

x := <- id // x will be 1

x := <- id // x will be 2

```
