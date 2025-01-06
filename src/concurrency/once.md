# `sync.Once` And Its Variants

## `sync.Once`

The `sync.Once` is exactly what you reach for when you need a function to
run just one time, no matter how many times it gets called or how many
goroutines hit it simultaneously.

It’s perfect for initializing a singleton resource, something that should
only happen once in your application’s lifecycle.

```go

//...

var once sync.Once
var conf Config

func GetConfig() Config {
    once.Do(func() {
        fmt.Println("fetchConfig called")
        conf = fetchConfig()
    })
    return conf
}

func main() {
    cfg1 := GetConfig()
    cfg2 := GetConfig()

    fmt.Println(cfg1 == cfg2)
}

// output:

// fetchConfig called
// true

```

After `sync.Once` successfully runs the function with `Do(f)`, it marks
itself as **complete** internally with a flag, and from that point on,
any further calls to `Do(f)` won’t run the function again, even if it’s
a different function.

This will make it difficult to deal with errors and panics during function
call.

## `sync.OnceValue`

`sync.OnceValue` returns a function that invokes `f` only once and returns the
value returned by `f`. The returned function may be called concurrently.

If `f` panics, the returned function will panic with the same value on
every call.

```go

func main() {
    once := sync.OnceValue(func() int {
        sum := 0
        for i := 0; i < 1000; i++ {
            sum += i
        }
        fmt.Println("Computed once:", sum)
        return sum
    })

    done := make(chan bool)
    for i := 0; i < 10; i++ {
        go func() {
            const want = 499500
            got := once()
            if got != want {
                fmt.Println("want", want, "got", got)
            }
            done <- true
        }()
    }
    
    for i := 0; i < 10; i++ {
        <-done
    }
}

```

## `sync.OnceValues`

`sync.OnceValues` works like `sync.OnceValue`, but lets you return multiple
values, including errors. So, you can cache both the result and any error
that comes up during the first run.

```go

var config Config

var getConfigOnce = sync.OnceValues(fetchConfig)

func main() {
  var err error

  config, err = getConfigOnce()
  if err != nil {
    log.Fatalf("Failed to fetch config: %v", err)
  }

 // ...
}

```

Now, `getConfigOnce()` behaves just like a normal function — it’s still
concurrent-safe, caches the result, and only incurs the cost of running the
function once.

Since `getConfigOnce()` returns the cached error the calling code needs to
be aware that it might be dealing with a cached error or failure. If you need
to retry, you’d have to create a new instance from `sync.OnceValues` to
re-run the initialization.
