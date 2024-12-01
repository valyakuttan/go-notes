# Basics

## Builtin functions

1. `append`

   The `append` built-in function appends elements to the end of a slice.
   If it has sufficient capacity, the destination is resliced to
   accommodate the new elements. If it does not, a new
   underlying array will be allocated. Append returns the updated
   slice. It is therefore necessary to store the result of
   `append`, often in the variable holding the slice itself.

   ```go

    slice = append(slice, elem1, elem2)
    slice = append(slice, anotherSlice...)

    // it is legal to append a string to a byte slice
    slice = append([]byte("hello "), "world"...)
   
   ```

2. `new`

   The `new` built-in function allocates memory. The first argument
   is a type, not a value, and the value returned is a pointer to a
   newly allocated zero value of that type.

   ```go
    
   type Person struct {
        Name     string
        Age      int
        Gender     string
   }

   p := new(Person) // p is a pointer to the Person type

   ```

3. `make`

   The make built-in function allocates and initializes an object of
   type slice, map, or chan (only). The first argument is a type,
   not a value. Unlike new, make's return type is the same as the
   type of its argument, not a pointer to it.

   ```go

   s := make([]int, 10, 15) // s is of type []int

   ```

## Type conversions

The expression `T(v)` converts the value `v` to the type `T`.

```go

i := 42
f := float64(i)
u := uint(f)

```
