# Timeout On Message Delay

The `time.After` function returns a channel that blocks for
the specified duration.

After the interval, the channel delivers the current time, once.

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    c := boring("Joe")
    for {
        select {
        case s := <-c:
            fmt.Println(s)
        case <-time.After(2 * time.Second):
            fmt.Println("You're too slow.")
            return
        }
    }
}

func boring(msg string) <-chan string { // Returns receive-only channel of strings.
    c := make(chan string)
    go func() { // We launch the goroutine from inside the function.
        for i := 0; ; i++ {
            c <- fmt.Sprintf("%s %d", msg, i)
            time.Sleep(time.Duration(rand.Intn(2e3)) * time.Millisecond)
        }
    }()
    return c // Return the channel to the caller.
}

```
