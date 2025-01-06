# Google Search

## Google Search 1.0

- The Google function takes a query and returns a slice of Results
  (which are just strings).

- Google invokes Web, Image, and Video searches serially, appending
  them to the results slice.

```go

package main

import (
    "fmt"
    "math/rand"
    "time"
)

type Result string

func Google(query string) (results []Result) {
    results = append(results, Web(query))
    results = append(results, Image(query))
    results = append(results, Video(query))
    return
}

var (
    Web = fakeSearch("web")
    Image = fakeSearch("image")
    Video = fakeSearch("video")
)

type Search func(query string) Result

func fakeSearch(kind string) Search {
        return func(query string) Result {
              time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
              return Result(fmt.Sprintf("%s result for %q\n", kind, query))
        }
}


func main() {
    rand.Seed(time.Now().UnixNano())
    start := time.Now()
    results := Google("golang") // HL
    elapsed := time.Since(start)
    fmt.Println(results)
    fmt.Println(elapsed)
}

```

## Google Search 2.0

- Run the Web, Image, and Video searches concurrently, and wait for
  all results.

- No locks. No condition variables. No callbacks.

```go

func Google(query string) (results []Result) {
    c := make(chan Result)
    go func() { c <- Web(query) } ()
    go func() { c <- Image(query) } ()
    go func() { c <- Video(query) } ()

    for i := 0; i < 3; i++ {
        result := <-c
        results = append(results, result)
    }
    return
}

```

## Google Search 2.1

- Don't wait for slow servers. No locks. No condition variables.
  No callbacks.

```go

func Google(query string) (results []Result) {
    c := make(chan Result)
    go func() { c <- Web(query) } ()
    go func() { c <- Image(query) } ()
    go func() { c <- Video(query) } ()

    timeout := time.After(80 * time.Millisecond)
    for i := 0; i < 3; i++ {
        select {
        case result := <-c:
            results = append(results, result)
        case <-timeout:
            fmt.Println("timed out")
            return
        }
    }
    return
}

```

## Avoid timeout

- How do we avoid discarding results from slow servers?

- Replicate the servers. Send requests to multiple replicas, and use the first response.

```go

package main

import (
    "fmt"
    "math/rand"
    "time"
)

type Result string
type Search func(query string) Result

func First(query string, replicas ...Search) Result {
    c := make(chan Result)
    searchReplica := func(i int) { c <- replicas[i](query) }
    for i := range replicas {
        go searchReplica(i)
    }
    return <-c
}

func main() {
    start := time.Now()
    result := First("golang",
        fakeSearch("replica 1"),
        fakeSearch("replica 2"))
    elapsed := time.Since(start)
    fmt.Println(result)
    fmt.Println(elapsed)
}

func fakeSearch(kind string) Search {
        return func(query string) Result {
              time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
              return Result(fmt.Sprintf("%s result for %q\n", kind, query))
        }
}

```

## Google Search 3.0

- Reduce tail latency using replicated search servers.

```go

package main

import (
    "fmt"
    "math/rand"
    "time"
)

type Result string
type Search func(query string) Result

var (
    Web1 = fakeSearch("web1")
    Web2 = fakeSearch("web2")

    Image1 = fakeSearch("image1")
    Image2 = fakeSearch("image2")

    Video1 = fakeSearch("video1")
    Video2 = fakeSearch("video2")
)

func Google(query string) (results []Result) {
    c := make(chan Result)
    go func() { c <- First(query, Web1, Web2) } ()
    go func() { c <- First(query, Image1, Image2) } ()
    go func() { c <- First(query, Video1, Video2) } ()

    timeout := time.After(80 * time.Millisecond)
    for i := 0; i < 3; i++ {
        select {
        case result := <-c:
            results = append(results, result)
        case <-timeout:
            fmt.Println("timed out")
            return
        }
    }
    return
}

func First(query string, replicas ...Search) Result {
    c := make(chan Result)
    searchReplica := func(i int) {
        c <- replicas[i](query)
    }
    for i := range replicas {
        go searchReplica(i)
    }
        return <-c
}

func fakeSearch(kind string) Search {
        return func(query string) Result {
              time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
              return Result(fmt.Sprintf("%s result for %q\n", kind, query))
        }
}

func main() {
    start := time.Now()
    results := Google("golang")
    elapsed := time.Since(start)
    fmt.Println(results)
    fmt.Println(elapsed)
}


```
