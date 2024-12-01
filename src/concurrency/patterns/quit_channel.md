# Signal quitting out of a conversation

When we're tired of listening to someone, we can tell them to stop talking.

```go

package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	quit := make(chan bool)
	c := boring("Joe", quit)
	for i := rand.Intn(20); i >= 0; i-- {
		fmt.Println(<-c)
	}
	quit <- true
}

func boring(msg string, quit chan bool) <-chan string { 
	// Returns receive-only channel of strings.
	c := make(chan string)
	go func() { // We launch the goroutine from inside the function.
		for i := 0; ; i++ {
			select {
			case c <- fmt.Sprintf("%s: %d", msg, i):
				time.Sleep(time.Duration(rand.Intn(2e3)) * time.Millisecond)
			case <-quit:
				return
			}
		}
	}()
	return c // Return the channel to the caller.
}

```

But the above model has a problem, we can't make sure that the `boring`
function is actually finished before we can quit, may be needed to do
some cleanup. This is not possible in the previous model because once
`main` goroutine is done nothing else can be done. So `main` must wait
until cleanup is over.

```go

package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	quit := make(chan string)
	c := boring("Joe", quit)
	for i := rand.Intn(20); i >= 0; i-- {
		fmt.Println(<-c)
	}
	quit <- "Bye!"
	fmt.Printf("Joe says: %q\n", <-quit)
}

func boring(msg string, quit chan string) <-chan string { 
	// Returns receive-only channel of strings.
	c := make(chan string)
	go func() { // We launch the goroutine from inside the function.
		for i := 0; ; i++ {
			select {
			case c <- fmt.Sprintf("%s: %d", msg, i):
				time.Sleep(time.Duration(rand.Intn(2e3)) * time.Millisecond)
			case <-quit:
				cleanup()
                quit <- "See you!"
                return
			}
		}
	}()
	return c // Return the channel to the caller.
}

func cleanup() {
	fmt.Println("Cleanup done!")
}

```
