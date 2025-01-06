# A Good Feed Reader Implementation

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

// A Subscription delivers Items over a channel.  Close cancels the
// subscription, closes the Updates channel, and returns the last fetch error,
// if any.
type Subscription interface {
    Updates() <-chan Item
    Close() error
}

// Fetch returns a Fetcher for Items from domain.
func Fetch(domain string) Fetcher {
    return fakeFetch(domain)
}

// Merge returns a Subscription that merges the item streams from subs.
// Closing the merged subscription closes subs.
func Merge(subs ...Subscription) Subscription {
    m := &merge{
        subs:    subs,
        updates: make(chan Item),
        quit:    make(chan struct{}),
        errs:    make(chan error),
    }
    for _, sub := range subs {
        go func(s Subscription) {
            for {
                var it Item
                select {
                case it = <-s.Updates():
                case <-m.quit:
                    m.errs <- s.Close()
                    return
                }

                select {
                case m.updates <- it:
                case <-m.quit:
                    m.errs <- s.Close()
                    return
                }
            }
        }(sub)
    }
    return m
}


// Subscribe returns a new Subscription that uses fetcher to fetch Items.
func Subscribe(fetcher Fetcher) Subscription {
    s := &sub{
        fetcher: fetcher,
        updates: make(chan Item),       // for Updates
        closing: make(chan chan error), // for Close
    }
    go s.loop()
    return s
}

func main() {
    // Subscribe to some feeds, and create a merged update stream.
    merged := Merge(
        Subscribe(Fetch("blog.golang.org")),
        Subscribe(Fetch("googleblog.blogspot.com")),
        Subscribe(Fetch("googledevelopers.blogspot.com")))

    // Close the subscriptions after some time.
    time.AfterFunc(3*time.Second, func() {
        fmt.Println("closed:", merged.Close())
    })

    // Print the stream.
    for it := range merged.Updates() {
        fmt.Println(it.Channel, it.Title)
    }

    // The loops are still running.  Let the race detector notice.
    time.Sleep(1 * time.Second)
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

// sub implements the Subscription interface.
type sub struct {
    fetcher Fetcher         // fetches items
    updates chan Item       // sends items to the user
    closing chan chan error // for Close
}

func (s *sub) Updates() <-chan Item {
    return s.updates
}

func (s *sub) Close() error {
    errc := make(chan error)
    s.closing <- errc
    return <-errc
}

// loop periodically fetches Items, sends them on s.updates, and exits
// when Close is called.  It extends dedupeLoop with logic to run
// Fetch asynchronously.
func (s *sub) loop() {
    var pending []Item
    var next time.Time
    var err error
    for {
        var fetchDelay time.Duration
        if now := time.Now(); next.After(now) {
            fetchDelay = next.Sub(now)
        }
        startFetch := time.After(fetchDelay) // enable fetch case

        var first Item
        var updates chan Item
        if len(pending) > 0 {
            first = pending[0]
            updates = s.updates // enable send case
        }
        select {
        case <-startFetch:
            var fetched []Item
            fetched, next, err = s.fetcher.Fetch()
            if err != nil {
                next = time.Now().Add(10 * time.Second)
            }
            pending = append(pending, fetched...)
        case errc := <-s.closing:
            errc <- err
            close(s.updates)
            return
        case updates <- first:
            pending = pending[1:]
        }
    }
}


type merge struct {
    subs    []Subscription
    updates chan Item
    quit    chan struct{}
    errs    chan error
}

func (m *merge) Updates() <-chan Item {
    return m.updates
}

func (m *merge) Close() (err error) {
    close(m.quit)
    for _ = range m.subs {
        if e := <-m.errs; e != nil {
            err = e
        }
    }
    close(m.updates)
    return
}

```
