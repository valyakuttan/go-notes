# Channels

In the boring examples we cheated: the `main` function couldn't see
the output from the other goroutine.

It was just printed to the screen, where we pretended we saw a conversation.

Real conversations require communication.

A **channel** in `Go` provides a connection between two goroutines,
allowing them to communicate.

```go

    // Declaring and initializing a bidirectional channel
    var c chan int
    c = make(chan int)
    // or
    c := make(chan int)

    // Sending on a channel.
    c <- 1

    // Receiving from a channel.
    // The "arrow" indicates the direction of data flow.
    value = <-c

    // create a receive only channel
    c1 := make(<-chan int)
    v := <-c1 // ok
    c1 <- 1 // Error

    // create a send only channel
    c2 := make(chan<- int)
    v := <-c2 // Error
    c2 <- 1 // Ok

```

We can use a channel to connect the `main` and `borring` goroutines
so that they can they can communicate.

```go

package main

import (
 "fmt"
 "math/rand"
 "time"
)


func main() {
    c := make(chan string)
    go boring("boring!", c)
    for i := 0; i < 5; i++ {
        fmt.Printf("You say: %q\n", <-c) // Receive expression is just a value.
    }
    fmt.Println("You're boring; I'm leaving.")
}

func boring(msg string, c chan string) {
    for i := 0; ; i++ {
        c <- fmt.Sprintf("%s %d", msg, i) // Expression to be sent can be any suitable value.
        time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
    }
}

```

## Synchronization

When the main function executes `<–c`, it will wait for a value to be sent.

Similarly, when the boring function executes `c <– value`, it waits for
a receiver to be ready.

A sender and receiver must both be ready to play their part in the
communication. Otherwise we wait until they are.

Thus channels both communicate and synchronize.
