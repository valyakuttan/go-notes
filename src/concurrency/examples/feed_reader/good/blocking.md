# 3. Fixing The Indefinite Blocking

The use of `select` statement avoids the blocking issue from the naive
implementation.

## 3.1 A Simple Solution Which Crashes

We cant send the fetched items, one at a time, as in the code below

```go

var pending []Item // appended by fetch; consumed by send
for {
    
    // ...

    select {
    //...
    case s.updates <- pending[0]:
        pending = pending[1:]
    }
}

```

This will crash when pending is empty.

## 3.2 Fixing The Panic When Pending Is Empty

By using the **nil channel trick** to enable send only when pending is non-empty,
we can fix the crashing.

```go

  var pending []Item // appended by fetch; consumed by send
    for {

        //...

        var first Item
        var updates chan Item
        if len(pending) > 0 {
            first = pending[0]
            updates = s.updates // enable send case
        }

        //...

        select {
            //...

        case updates <- first:
            pending = pending[1:]
            
            //...
        }
    }

```
