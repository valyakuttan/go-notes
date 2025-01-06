# A Naive Feed Reader Implementation

```go

package main

import (
    "fmt"
    "math/rand"
    "time"
)

// An Item is a stripped-down RSS item.
type Item struct{ Title, Channel, GUID string }

// A Fetcher fetches Items and returns the time when the next fetch should be
// attempted.  On failure, Fetch returns a non-nil error.
type Fetcher interface {
    Fetch() (items []Item, next time.Time, err error)
}

// Fetch returns a Fetcher for Items from domain.
func Fetch(domain string) Fetcher {
    return fakeFetch(domain)
}

// A Subscription delivers Items over a channel.  Close cancels the
// subscription, closes the Updates channel, and returns the last fetch error,
// if any.
type Subscription interface {
    Updates() <-chan Item
    Close() error
}

func Subscribe(fetcher Fetcher) Subscription {
    return NaiveSubscribe(fetcher)
}

func Merge(subs ...Subscription) Subscription {
    return NaiveMerge(subs...)
}

func main() {
    // Subscribe to some feeds, and create a merged update stream.
    merged := Merge(
        Subscribe(Fetch("blog.golang.org")),
        Subscribe(Fetch("googleblog.blogspot.com")),
        Subscribe(Fetch("googledevelopers.blogspot.com")))

    // Close the subscriptions after some time.
    time.AfterFunc(3*time.Second, func() {
        merged.Close()
        fmt.Println("closed all subscriptions")
    })

    // Print the stream.
    for it := range merged.Updates() {
        fmt.Println(it.Channel, it.Title)
    }

    // The loops are still running.  Let the race detector notice.
    time.Sleep(1 * time.Second)

    panic("show me the stacks")
}

// naiveMerge is a version of Merge that doesn't quite work right.  In
// particular, the goroutines it starts may block forever on m.updates
// if the receiver stops receiving.
type naiveMerge struct {
    subs    []Subscription
    updates chan Item
}

func (m *naiveMerge) Close() (err error) {
    for _, sub := range m.subs {
        if e := sub.Close(); err == nil && e != nil {
            err = e
        }
    }
    close(m.updates)
    return
}

func (m *naiveMerge) Updates() <-chan Item {
    return m.updates
}

func NaiveMerge(subs ...Subscription) Subscription {
    m := &naiveMerge{
        subs:    subs,
        updates: make(chan Item),
    }
    for _, sub := range subs {
        go func(s Subscription) {
            for it := range s.Updates() {
                m.updates <- it
            }
        }(sub)
    }
    return m
}

func fakeFetch(domain string) Fetcher {
    return &fakeFetcher{channel: domain}
}

type fakeFetcher struct {
    channel string
    items   []Item
}

// FakeDuplicates causes the fake fetcher to return duplicate items.
var FakeDuplicates bool

func (f *fakeFetcher) Fetch() (items []Item, next time.Time, err error) {
    now := time.Now()
    next = now.Add(time.Duration(rand.Intn(5)) * 500 * time.Millisecond)
    item := Item{
        Channel: f.channel,
        Title:   fmt.Sprintf("Item %d", len(f.items)),
    }
    item.GUID = item.Channel + "/" + item.Title
    f.items = append(f.items, item)
    if FakeDuplicates {
        items = f.items
    } else {
        items = []Item{item}
    }
    return
}

func NaiveSubscribe(fetcher Fetcher) Subscription {
    s := &naiveSub{
        fetcher: fetcher,
        updates: make(chan Item),
    }
    go s.loop()
    return s
}

type naiveSub struct {
    fetcher Fetcher
    updates chan Item
    closed  bool
    err     error
}

func (s *naiveSub) Updates() <-chan Item {
    return s.updates
}

func (s *naiveSub) loop() {
    for {
        if s.closed {
            close(s.updates)
            return
        }
        items, next, err := s.fetcher.Fetch()
        if err != nil {
            s.err = err
            time.Sleep(10 * time.Second)
            continue
        }
        for _, item := range items {
            s.updates <- item
        }
        if now := time.Now(); next.After(now) {
            time.Sleep(next.Sub(now))
        }
    }
}

func (s *naiveSub) Close() error {
    s.closed = true
    return s.err
}

```
