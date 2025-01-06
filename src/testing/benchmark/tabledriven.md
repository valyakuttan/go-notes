# Table Driven Benchmarks

Just like normal tests we can do benchmarks in a table driven fashion.

```go

func BenchmarkFooer(b *testing.B) {
    // Defining the columns of the table
    var benchmarks = []struct {
        name  string
        input int
    }{
        {"9", 9},
        {"3", 3},
        {"1", 1},
        {"0", 0},
    }
    // The execution loop
    for _, bm := range benchmarks {
        b.Run(bm.name, func(b *testing.B) {
            for i := 0; i < b.N; i++ {
                Fooer(bm.input)
            }
        })
    }
}

```
