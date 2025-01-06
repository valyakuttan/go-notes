# 2D Slices

Sometimes it's necessary to allocate a 2D slice, a situation that can arise when
processing scan lines of pixels, for instance. There are two ways to achieve this.

1. Allocate each slice independently;

2. Allocate a single array and point the individual slices into it.

Which to use depends on your application.

If the slices might grow or shrink, they should be allocated independently to avoid
overwriting the next line; if not, it can be more efficient to construct the object
with a single allocation.

```go

func independentAlloc(XSize, YSize int) [][]uint8 {
 // Allocate the top-level slice.
 picture := make([][]uint8, YSize) // One row per unit of y.

 // Loop over the rows, allocating the slice for each row.
 for i := range picture {
  picture[i] = make([]uint8, XSize)
 }
 
 return picture
}

func blocAlloc(XSize, YSize int) [][]uint8 {
 // Allocate the top-level slice, the same as before.
 picture := make([][]uint8, YSize) // One row per unit of y.
 
 // Allocate one large slice to hold all the pixels.
 pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
 
 // Loop over the rows, slicing each row from the front of the remaining pixels slice.
 for i := range picture {
  picture[i], pixels = pixels[:XSize], pixels[XSize:]
 }
 
 return picture
}


```
