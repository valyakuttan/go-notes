# Writing Benchmark Tests

Benchmark tests are a way of testing your code performance. The goal of
those tests is to verify the runtime and the memory usage of an algorithm
by running the same function many times.

To create a benchmark test:

- Your test function needs to be in a `*_test` file.

- The name of the function must start with `Benchmark`.

- The function must accept `*testing.B` as the unique parameter.

- The test function must contain a for loop using `b.N` as its upper bound.

```go

func BenchmarkFooer(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Fooer(i)
    }
}

```

The target code should be run `N` times in the benchmark function, and `N` is
automatically adjusted at runtime until the execution time of each iteration
is statistically stable.

The benchmark tests can be performed by  using `-bench=regexp` flag.

```go

    go test -bench=. # will run all benchmarks

```

If you want to gather memory allocation statistics for each benchmark
 use `-benchmem` flag.

```go

    go test -bench=. -benchmem

```

You can run benchmarks multiple times using the -count flag

```go

    go test -bench=. -benchmem -count=10

```
