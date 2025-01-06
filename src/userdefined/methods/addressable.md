# Go Addressable Types

An addressable type has a real memory address which will not change
while its value changes. A non-addressable type is one that does not
have an address, and can change while the value evolves.

Examples of Addressable types include basic types like `int64`, string,
struct etc.

Non-Addressable types include **maps**, **functions**, **interfaces**.
