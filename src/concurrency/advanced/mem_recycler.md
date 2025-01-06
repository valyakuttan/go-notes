# Memory Recyler

## 1. Recycler Without Channel

```go

func recycler(give <-chan []byte, get chan<- []byte) {
    q := new(list.List)

    for {
        if q.Len() == 0 {
            q.PushFront(make([]byte, 100))
        }

        e := q.Front()

        select {
        case s := <-give:
            q.PushFront(s[:0])
        
        case get <- e.Value.([]byte):
            q.Remove(e)
         }
    }
}

```
