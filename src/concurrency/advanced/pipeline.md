# Pipelines and cancellation

A **pipeline** is a series of stages connected by channels, where each
stage is a group of goroutines running the same function. In each stage,
the goroutines

- receive values from upstream via inbound channels

- perform some function on that data, usually producing new values

- send values downstream via outbound channels

Each stage has any number of inbound and outbound channels, except the first
and last stages, which have only outbound or inbound channels, respectively.
The first stage is sometimes called the **source** or producer; the last
stage, the **sink** or consumer.

## A Simple Pipeline

Consider a pipeline with three stages.

### Stage 1. Generator

This emits a sequence of numbers on a channel

```go

func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}

```

### 2. Square

```go

func sq(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}

```

### 3. Main

```go

unc main() {
    // Set up the pipeline.
    c := gen(2, 3)
    out := sq(c)

    // Consume the output.
    fmt.Println(<-out) // 4
    fmt.Println(<-out) // 9
}

```

## A Pipeline With Multiple Instances Of A Stage

With the help of a merge stage we can setup a pipe which has some stages with
multiple instances.

```go

func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(in)
    c2 := sq(in)

    // Consume the merged output from c1 and c2.
    for n := range merge(c1, c2) {
        fmt.Println(n) // 4 then 9, or 9 then 4
    }
}

func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed, then calls wg.Done.
    output := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // Start a goroutine to close out once all the output goroutines are
    // done.  This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}

```

## Pipelines With Cancellation

In our example pipeline, if a stage fails to consume all the inbound values,
the goroutines attempting to send those values will block indefinitely:

```go

    // Consume the first value from the output.
    out := merge(c1, c2)
    fmt.Println(<-out) // 4 or 9
    return
    // Since we didn't receive the second value from out,
    // one of the output goroutines is hung attempting to send it.

```

This is a resource leak: goroutines consume memory and runtime resources,
and heap references in goroutine stacks keep data from being garbage
collected. **Goroutines** are not **not garbage collected**; they must
**exit on their own**.

We need a way to tell an unknown and unbounded number of goroutines to stop
sending their values downstream. In **Go**, we can do this by closing a
channel, because a receive operation on a closed channel can always proceed
immediately, yielding the element type’s zero value.

This means that main can unblock all the senders simply by closing a channel
called **done**. This close is effectively a broadcast signal to the senders.

We extend each of our pipeline functions to accept done as a parameter and
arrange for the close to happen via a **defer** statement, so that all return
paths from **main** will signal the pipeline stages to exit.

```go

func main() {
    // Set up a done channel that's shared by the whole pipeline,
    // and close that channel when this pipeline exits, as a signal
    // for all the goroutines we started to exit.
    done := make(chan struct{})
    defer close(done)

    in := gen(done, 2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(done, in)
    c2 := sq(done, in)

    out := merge(done, c1, c2)

    // Consume the first value from output.
    fmt.Println(<-out)

    // done will be closed by the deferred call.
}

```

Each of our pipeline stages is now free to return as soon as `done` is closed.
The output routine in `merge` can return without draining its inbound channel,
since it knows the upstream sender, `sq`, will stop attempting to send when
`done` is closed. output ensures `wg.Done` is called on all return paths
via a `defer` statement.

```go

func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    wg.Add(len(cs))
    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c or done is closed, then calls
    // wg.Done.
    output := func(c <-chan int) {
        defer wg.Done()
        for n := range c {
            select {
            case out <- n:
            case <-done:
                return
            }
        }
    }

    for _, c := range cs {
        go output(c)
    }

    // Start a goroutine to close out once all the output goroutines are
    // done.  This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()

    return out
}

```

Similarly, `sq` and `gen` can return as soon as `done` is closed.
`sq` ensures its out channel is closed on all return paths via
a `defer` statement.

```go

func sq(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-done:
                return
            }
        }
    }()
    return out
}

func gen(done <-chan struct{}, nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            select {
            case out <- n:
            case <-done:
                return
            }
        }
        close(out)
    }()
    return out
}

```

## Guidelines For Pipeline Construction

- stages close their outbound channels when all the send operations are done.

- stages keep receiving values from inbound channels until those channels
  are closed or the senders are unblocked.

Pipelines unblock senders either by ensuring there’s enough buffer for all
the values that are sent or by explicitly signalling senders when the
receiver may abandon the channel.
