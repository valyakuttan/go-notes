# Function Values and Closures

## Function Values

Functions are values too. They can be passed around just like other values.

Function values may be used as function arguments and return values.

```go

func compute(fn func(float64, float64) float64) float64 {
 return fn(3, 4)
}

func main() {
 hypot := func(x, y float64) float64 {
  return math.Sqrt(x*x + y*y)
 }

 fmt.Println(hypot(5, 12))

 fmt.Println(compute(hypot))

 fmt.Println(compute(math.Pow))
}

```

## Closures

Go functions may be closures. A closure is a function value that references
variables from outside its body. The function may access and assign to the
referenced variables; in this sense the function is "bound" to the variables.

For example, the adder function returns a closure. Each closure is bound to its own sum variable.

```go


func adder() func(int) int {
 sum := 0
 return func(x int) int {
  sum += x
  return sum
 }
}

func main() {
 pos, neg := adder(), adder()

 for i := 0; i < 10; i++ {
  fmt.Printf("(%d, %d), ", pos(i), neg(-2*i))
  // (0,0), (1, -2), .., (10, -20)
 }

}

```

## Named Return Parameters

- It serves as documentation.

- They are auto-declared and initialized to the zero values.

- If you have multiple return sites, you don't need to change
  them all if you change the function's return values since
  it will just say `return`.

```go

func nextInt(b []byte, pos int) (value, nextPos int) {
    // ...
}

```
