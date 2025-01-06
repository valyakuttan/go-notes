# Terminate Workers By Closing A Channel

```go

func worker(die chan bool) {
    for {
        select {
            
        //... do stuff cases

        case <-die:
            return
        }
    }

}

func main() {
    die := make(chan bool)

    for i := 0; i < 1000; i++ {
        go worker(die)
    }

    //... workers are waiting for signal to stop
    
    close(die)

    //...all workers terminate

    //...
}

```
