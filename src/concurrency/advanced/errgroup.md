# Package `errgroup`

Package `errgroup` provides synchronization, error propagation, and Context
cancelation for groups of goroutines working on subtasks of a common task.

## `Group`

A `Group` is a collection of goroutines working on subtasks that are part of
the same overall task. A zero Group is valid, has no limit on the number of
active goroutines, and does not cancel on error.

## Using `Group` To Simplify Goroutine Counting and Error Handling

```go

func main() {
    g := new(errgroup.Group)
    var urls = []string{
        "http://www.golang.org/",
        "http://www.google.com/",
        "http://www.somestupidname.com/",
    }
    for _, url := range urls {
        // Launch a goroutine to fetch the URL.
        url := url
        g.Go(func() error {
            // Fetch the URL.
            resp, err := http.Get(url)
            if err == nil {
                resp.Body.Close()
            }
            return err
        })
    }
    // Wait for all HTTP fetches to complete.
    if err := g.Wait(); err == nil {
        fmt.Println("Successfully fetched all URLs.")
    }
}

```

## Using Goroutine To Synchronize Parallel Task

```go

var (
    Web   = fakeSearch("web")
    Image = fakeSearch("image")
    Video = fakeSearch("video")
)

type Result string
type Search func(ctx context.Context, query string) (Result, error)

func fakeSearch(kind string) Search {
    return func(_ context.Context, query string) (Result, error) {
        return Result(fmt.Sprintf("%s result for %q", kind, query)), nil
    }
}

func main() {
    Google := func(ctx context.Context, query string) ([]Result, error) {
        g, ctx := errgroup.WithContext(ctx)

        searches := []Search{Web, Image, Video}
        results := make([]Result, len(searches))
        for i, search := range searches {
            i, search := i, search
            g.Go(func() error {
                result, err := search(ctx, query)
                if err == nil {
                    results[i] = result
                }
                return err
            })
        }
        if err := g.Wait(); err != nil {
            return nil, err
        }
        return results, nil
    }

    results, err := Google(context.Background(), "golang")
    if err != nil {
        fmt.Fprintln(os.Stderr, err)
        return
    }
    for _, result := range results {
        fmt.Println(result)
    }

}


```

## Using `Group` To Implement Multi-stage Pipeline

```go

// Pipeline demonstrates the use of a Group to implement a multi-stage
// pipeline: a version of the MD5All function with bounded parallelism from
// https://blog.golang.org/pipelines.
func main() {
    m, err := MD5All(context.Background(), ".")
    if err != nil {
        log.Fatal(err)
    }

    for k, sum := range m {
        fmt.Printf("%s:\t%x\n", k, sum)
    }
}

type result struct {
    path string
    sum  [md5.Size]byte
}

// MD5All reads all the files in the file tree rooted at root and returns a map
// from file path to the MD5 sum of the file's contents. If the directory walk
// fails or any read operation fails, MD5All returns an error.
func MD5All(ctx context.Context, root string) (map[string][md5.Size]byte, error) {
    // ctx is canceled when g.Wait() returns. When this version of MD5All returns
    // - even in case of error! - we know that all of the goroutines have finished
    // and the memory they were using can be garbage-collected.
    g, ctx := errgroup.WithContext(ctx)
    paths := make(chan string)

    g.Go(func() error {
        defer close(paths)
        return filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if !info.Mode().IsRegular() {
                return nil
            }
            select {
            case paths <- path:
            case <-ctx.Done():
                return ctx.Err()
            }
            return nil
        })
    })

    // Start a fixed number of goroutines to read and digest files.
    c := make(chan result)
    const numDigesters = 20
    for i := 0; i < numDigesters; i++ {
        g.Go(func() error {
            for path := range paths {
                data, err := os.ReadFile(path)
                if err != nil {
                    return err
                }
                select {
                case c <- result{path, md5.Sum(data)}:
                case <-ctx.Done():
                    return ctx.Err()
                }
            }
            return nil
        })
    }
    go func() {
        g.Wait()
        close(c)
    }()

    m := make(map[string][md5.Size]byte)
    for r := range c {
        m[r.path] = r.sum
    }
    // Check whether any of the goroutines failed. Since g is accumulating the
    // errors, we don't need to send them (or check for them) in the individual
    // results sent on the channel.
    if err := g.Wait(); err != nil {
        return nil, err
    }
    return m, nil
}


```
