# Select Statement

Select is a control structure unique to concurrency.

The `select` statement provides another way to handle multiple channels.

It's like a `switch`, but each case is a communication:

- All channels are evaluated.

- Selection blocks until one communication can proceed, which then does.

- If multiple can proceed, `select` chooses pseudo-randomly.

- A `default` clause, if present, executes immediately if no channel is ready.

```go

package main

import (
	"fmt"
	"math/rand"
	"time"
)


func main() {
	c1 := make(chan int)
	c2 := make(chan int)
	c3 := make(chan int)

	go func() {
		time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
		c1 <- 100
	}()

	go func() {
		time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
		c2 <- 200
	}()

	go func() {
		time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
		<-c3
	}()

	done := make(chan int)
	go func() {
		count := 0
		for {
			select {
			case v1 := <-c1:
				fmt.Printf("received %v from c1\n", v1)
				count++
			case v2 := <-c2:
				fmt.Printf("received %v from c2\n", v2)
				count++
			case c3 <- 300:
				fmt.Printf("sent %v to c3\n", 300)
				count++
			default:
				time.Sleep(time.Duration(rand.Intn(2e3)) * time.Millisecond)
				if count < 2 {
					fmt.Printf("no one was ready to communicate\n")
				} else {
					fmt.Printf("leaving\n")
					done <- 1
				}
			}
		}
	}()
	<-done
}


```
