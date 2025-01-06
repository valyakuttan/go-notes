# Useful Features which Support Developers

## Deadlock detection

When all the **goroutines** are in **deadlock**. The `main` will `panic`.

## Stack traces

Calling `panic` will print the **stack traces** of all **goroutines** which
is **blocked** at the moment of call to `panic`.

This is useful to check whether there is any **memory leaks**.
Long-lived programs need to clean up.
