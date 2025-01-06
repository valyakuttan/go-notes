# Share Memory By Communicating

## An Example Ping-Pong

- We consider each turn of a player as taking ownership of the `Ball`

- We model`table` as a channel to transfer the ownership of `Ball`.

- By designing the program around the **ownership** model, we make sure
  that only the owner is allowed to manipulate the `Ball` data structure.

```go

package main

import (
    "fmt"
    "time"
)

type Ball struct{ hits int }

func main() {
    table := make(chan *Ball)
    go player("ping", table)
    go player("pong", table)

    table <- new(Ball) // game on; toss the ball
    time.Sleep(1 * time.Second)
    <-table // game over; grab the ball
}

func player(name string, table chan *Ball) {
    for {
        ball := <-table
        ball.hits++
        fmt.Println(name, ball.hits)
        time.Sleep(100 * time.Millisecond)
        table <- ball
    }
}


```
