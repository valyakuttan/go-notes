# WaitGroup

A `WaitGroup` waits for a collection of goroutines to finish. The main
goroutine calls `WaitGroup.Add` to set the number of goroutines to wait
for. Then each of the goroutines runs and calls `WaitGroup.Done` when
finished. At the same time, `WaitGroup.Wait` can be used to block until
all goroutines have finished.

```go

type httpPkg struct{}

func (httpPkg) Get(url string) {}

var http httpPkg

func main() {
    var wg sync.WaitGroup
    var urls = []string{
        "http://www.golang.org/",
        "http://www.google.com/",
        "http://www.example.com/",
    }
    for _, url := range urls {
        // Increment the WaitGroup counter.
        wg.Add(1)
        // Launch a goroutine to fetch the URL.
        go func(url string) {
            // Decrement the counter when the goroutine completes.
            defer wg.Done()
            // Fetch the URL.
            http.Get(url)
        }(url)
    }
    // Wait for all HTTP fetches to complete.
    wg.Wait()
}


```
