# Slice Tricks

## Make Copy Of A Slice

```go

b := make([]T, len(a))
copy(b, a)

```

## Cut Part Of A Slice

```go

a = append(a[:i], a[j:]...) // a[i:j-1] removed from a

```

## Reverse A Slice

To replace the contents of a slice with the same elements but in reverse order:

```go

for i := len(a)/2-1; i >= 0; i-- {
    opp := len(a)-1-i
    a[i], a[opp] = a[opp], a[i]
}

```

## Filter Contents Of A Slice

```go

n := 0
for _, x := range a {
    if keep(x) {
        a[n] = x
        n++
    }
}
a = a[:n]

```
