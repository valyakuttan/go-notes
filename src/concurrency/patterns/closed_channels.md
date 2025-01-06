# Properties Of Closed Channels

## 1. Closed Channels Never Block

```go

    //...

    c := make(chan bool)
    close(c)
    x := <-c
    fmt.Printf("%#v\n", x) // print zero value of channel's type

    //...

     c := make(chan bool)
    close(c)
    x, ok := <-c
    fmt.Printf("%#v %#v\n", x, ok) // ok is false

    //...
    c := make(chan bool)
    close(c)
    c <- true // crashes the program

```

## 2. Closing Buffered Channels

```go

//...

c := make(chan int 3)
c <- 1
c <- 2
c <- 3
close(c)

fmt.Printf("%d\n", <-c) // 1
fmt.Printf("%d\n", <-c) // 2
fmt.Printf("%d\n", <-c) // 3

fmt.Printf("%d\n", <-c) // 0

```
