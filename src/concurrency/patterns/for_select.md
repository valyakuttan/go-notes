# Common Idiom: for/select

```go

//...

    for {
        
        //...

        select {
        case x := <- somechan:

            //... do stuff with x
        
        case y, not_closed := <- otherchan:

            //... do stuff with x

            //... check not_closed to see if the
            //... channel is closed

        case output <- z:
            //.. z was sent
        
        default:
            //... no one wants to communicate

        }

        //...
    }

//...

```
