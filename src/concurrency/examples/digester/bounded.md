# A Bounded Implementation

The `MD5All` implementation in starts a new goroutine for each file. In a
directory with many large files, this may allocate more memory than is
available on the machine.

We can limit these allocations by bounding the number of files read in
parallel. Here we are creating a fixed number of goroutines for reading
files. Our pipeline now has three stages:

## 1. Walk The Tree

The first stage, `walkFiles`, emits the paths of regular files in the tree

```go

func walkFiles(done <-chan struct{}, root string) (<-chan string, <-chan error) {
    paths := make(chan string)
    errc := make(chan error, 1)
    go func() {
        // Close the paths channel after Walk returns.
        defer close(paths)
        // No select needed for this send, since errc is buffered.
        errc <- filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if !info.Mode().IsRegular() {
                return nil
            }
            select {
            case paths <- path:
            case <-done:
                return fmt.Errorf("walk canceled")
            }
            return nil
        })
    }()
    return paths, errc
}

```

## 2. Read And Digest The Files

The middle stage starts a fixed number of digester goroutines that receive
file names from paths and send results on channel `c`

```go


// A result is the product of reading and summing a file using MD5.

type result struct {
    path string
    sum  [md5.Size]byte
    err  error
}

func digester(done <-chan struct{}, paths <-chan string, c chan<- result) {
    for path := range paths {
        data, err := os.ReadFile(path)
        select {
        case c <- result{path, md5.Sum(data), err}:
        case <-done:
            return
        }
    }
}

```

## 3. Collect The Digests

The final stage receives all the results from `c` then checks the error from
`errc`. This check cannot happen any earlier, since before this point,
`walkFiles` may block sending values downstream.

```go

// MD5All reads all the files in the file tree rooted at root and returns a map
// from file path to the MD5 sum of the file's contents.  If the directory walk
// fails or any read operation fails, MD5All returns an error.  In that case,
// MD5All does not wait for inflight read operations to complete.

func MD5All(root string) (map[string][md5.Size]byte, error) {
    // MD5All closes the done channel when it returns; it may do so before
    // receiving all the values from c and errc.

    done := make(chan struct{})
    defer close(done)

    paths, errc := walkFiles(done, root)

    // Start a fixed number of goroutines to read and digest files.
    c := make(chan result) //
    var wg sync.WaitGroup
    const numDigesters = 20

    wg.Add(numDigesters)

    for i := 0; i < numDigesters; i++ {
        go func() {
            digester(done, paths, c)
            wg.Done()
        }()

    }

    go func() {
        wg.Wait()
        close(c)

    }()

    // End of pipeline.
    m := make(map[string][md5.Size]byte)
    for r := range c {
        if r.err != nil {
            return nil, r.err
        }
        m[r.path] = r.sum
    }

    // Check whether the Walk failed.
    if err := <-errc; err != nil {
        return nil, err

    }

    return m, nil
}

```

## The `main` Goroutine

```go

//go:build ignore

package main

import (
    "crypto/md5"
    "fmt"
    "os"
    "path/filepath"
    "sort"
    "sync"
)

func main() {

    // Calculate the MD5 sum of all files under the specified directory,
    // then print the results sorted by path name.
    m, err := MD5All(os.Args[1])

    if err != nil {
        fmt.Println(err)
        return
    }

    var paths []string
    for path := range m {
        paths = append(paths, path)
    }

    sort.Strings(paths)
    for _, path := range paths {
        fmt.Printf("%x  %s\n", m[path], path)
    }
}
```
