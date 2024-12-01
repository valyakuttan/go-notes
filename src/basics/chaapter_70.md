# Defer Statement

Go's defer statement schedules a function call (the deferred function)
to be run immediately before the function executing the defer returns.
It's an unusual but effective way to deal with situations such as
resources that must be released regardless of which path a function
takes to return. The canonical examples are unlocking a **mutex** or
 closing a file.

 ```go

// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}

 ```

 The arguments to the deferred function (which include the receiver if the
 function is a method) are evaluated when the `defer` executes, not when
 the call executes.
