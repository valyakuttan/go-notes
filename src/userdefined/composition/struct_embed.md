# Struct Embedding

Including an anonymous field in a `struct` is known as **embedding**.
The main purpose of type embedding is to extend the functionalities of
the embedded types into the embedding type, so that we don't need
to re-implement the functionalities of the embedded types for the
embedding type. Embedding promotes the methods defined for the
embedded type into the embedding type.

```go

type Person struct {
    Name string
    Age  int
}

func (p Person) Details() string {
    return fmt.Sprintf("Name: %s, Age: %d", p.Name, p.Age)
}

type Employee struct {
    Person
    EmployeeID string
}

func main() {
    p := Person{"foo", 25}

    e := Employee{p, "string"}

    fmt.Println(e.Details()) // call e.p.Details()
}

```
