# Generators

Generators are functions which return a channel.

Since channels are first-class values in Go, generators can be used
to create channels.

```go

package main

import (
 "fmt"
 "math/rand"
 "time"
)

func main() {
    joe := Generator("Joe")
    ann := Generator("Ann")

    for i := 0; i < 5; i++ {
    // because of the synchronized nature of channels,
    // the order of printing is
    // preserved, which many not always desirable 
    fmt.Printf("You say: %q\n", <-joe)
    fmt.Printf("You say: %q\n", <-ann)

    }

    fmt.Println("You're both boring; I'm leaving ")

}

func boring(msg string) <-chan string { // Returns receive-only channel of strings.
    c := make(chan string)
    go func() { // We launch the goroutine from inside the function.
        for i := 0; ; i++ {
            c <- fmt.Sprintf("%s %d", msg, i)
            time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
        }
    }()
    return c // Return the channel to the caller.
}

```
