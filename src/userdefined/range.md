# Range Clause

A range clause provides a way to iterate over an array, slice,
string, map, or channel.

```go

// iterates over [0,5)
 for i := range 5 {
  fmt.Print(i, " ") // 0,1,..,4
 }

 myMap := map[int]string{1: "one", 2: "two"}

 // iterates over (key, value) pair
 for k, v := range myMap {
  fmt.Printf("key=%v, value=%v", k, v)
 }

 // iterates over (index, value) pair
 for i, v := range []int{1, 2, 3, 4, 5} {
  fmt.Printf("array value at [%d]=%v", i, v)
 }

```

## Gotchas

 When iterating over a slice or map of values, one might try this:

```go

 items_bad := make([]map[int]int, 10)
 
 for _, item := range items_bad {
  item = make(map[int]int, 1) // Oops! item is only a copy of the slice element.
  item[1] = 2                 // This 'item' will be lost on the next iteration.
 }

```

The make and assignment look like they might work, but the value property of
range (stored here as item) is a copy of the value from items, not a pointer
to the value in items.

 This will work:

 ```go
 
 items_good := make([]map[int]int, 10)

 for i := range items_good {
  items_good[i] = make(map[int]int, 1)
  items_good[i][1] = 2
 }
 
 ```
