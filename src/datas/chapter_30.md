# Maps

A map maps keys to values.

The zero value of a map is nil. A nil map has no keys, nor can keys be added.

The `make` function returns a map of the given type, initialized and
ready for use.

## Mutating Maps

```go
// Insert or update an element
m[key] = elem 

// Retrieve an element
elem = m[key] 

// Delete an element:
delete(m, key)

// Test that a key is present with a two-value assignment:
elem, ok := m[key]

```
