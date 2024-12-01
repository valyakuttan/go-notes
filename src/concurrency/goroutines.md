# Goroutines

A goroutine is an independently executing function, launched by a
`go` statement. It has its own call stack, which grows and
shrinks as required. It's very cheap. It's practical to have
thousands, even hundreds of thousands of goroutines.

It's not a thread. There might be only one thread in a program
with thousands of goroutines.

Goroutines are multiplexed dynamically onto threads as needed
to keep all the goroutines running.

```go

package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    boring("boring!")
    fmt.Println("I'm listening.")

    // When main returns, the program exits and takes
    // the boring function down with it.
    // So if there is no waiting, nothing will get printed
    time.Sleep(2 * time.Second)
    fmt.Println("You're boring; I'm leaving.")
}

func boring(msg string) {
    for i := 0; ; i++ {
        fmt.Println(msg, i)
        time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
    }
}

```

The `go` statement runs the function as usual, but doesn't make the caller wait.

It launches a goroutine.

The functionality is analogous to the `&` on the end of a shell command.
