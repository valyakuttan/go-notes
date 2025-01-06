# 3. Indefinite Blocking

The sends over a channel will block until a goroutine is ready to receive
the value. So if a client close a subscription and stop receiving items.Then
the  send over `s.updates` will block `loop` forever.

```go

func (s *naiveSub) loop() {
    for {
        
        //...

        for _, item := range items {
            s.updates <- item
        }

        //...
    }
}

//...

```
