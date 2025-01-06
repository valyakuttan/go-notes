
# The `Fetcher` Interface And Fetch Function

The `Fetcher` interface is used to interact with the **Rss Reader** provided
by the external package.

The `Fetch` function takes a domain and return a `Fetcher` implementation.

```go

type Fetcher interface {
    Fetch() (items []Item, next time.Time, err error)
}

func Fetch(domain string) Fetcher {...} // fetches Items from domain

```
