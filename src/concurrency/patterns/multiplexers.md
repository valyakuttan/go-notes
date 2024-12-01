# Multiplexers

In the `Generators` example, `Joe` and `Ann` count in
 **lockstep**. Even if `Ann` is ready before `Joe`, she
 must wait for `Joe` to speak. This may not be always
 desirable.

```go

   joe := Generator("Joe")
    ann := Generator("Ann")

    for i := 0; i < 5; i++ {
    // because of the synchronized nature of channels,
    // the order of printing is
    // preserved, which many not always desirable 
    fmt.Printf("You say: %q\n", <-joe)
    fmt.Printf("You say: %q\n", <-ann)

    }

```

We can instead use a fan-in function to let whosoever is ready talk.

```go

package main

import (
 "fmt"
 "math/rand"
 "time"
)

func main() {
    c := fanIn(boring("Joe"), boring("Ann"))
    for i := 0; i < 10; i++ {
        fmt.Println(<-c)
    }
    fmt.Println("You're both boring; I'm leaving.")
}

func fanIn(input1, input2 <-chan string) <-chan string {
    c := make(chan string)
    go func() { for { c <- <-input1 } }()
    go func() { for { c <- <-input2 } }()
    return c
}

```
