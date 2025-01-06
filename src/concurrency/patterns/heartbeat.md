# Heartbeat

```go

func worker(start chan bool) {
    heartbeat := time.NewTicker(100 * time.Millisecond)

    for {
        select {
        //... do some stuff

        case <-heartbeat.C:
            //... heartbeat stuff
        }
    }
}

```
