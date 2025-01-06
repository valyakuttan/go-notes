# Unit Testing

Unit tests in **Go** are located in the same package (that is, the same
folder) as the tested function. By convention, if your function is in the
file `fooer.go` file, then the unit test for that function is in the file
`fooer_test.go`.

```go

package main

import "strconv"

// If the number is divisible by 3, write "Foo" otherwise, the number
func Fooer(input int) string {

    isfoo := (input % 3) == 0

    if isfoo {
        return "Foo"
    }

    return strconv.Itoa(input)
}

```

```go

package main
import "testing"
func TestFooer(t *testing.T) {
    result := Fooer(3)
    if result != "Foo" {
    t.Errorf("Result was incorrect, got: %s, want: %s.", result, "Foo")
    }
}

```
