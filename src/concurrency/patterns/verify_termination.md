# Verify Termination

We can terminate a goroutine and verify the termination with
`for/select` idiom.

```go

func worker(die chan bool) {
    for {
        select {
            
        //... do stuff cases

        case <-die:
            //... do cleanup

            die <- ture
            return
        }
    }

}

func main() {
    die := make(chan bool)
    go worker(die)

    //...

    die <- true
    <- die
}

```
