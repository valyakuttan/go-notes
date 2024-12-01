# A Simple Load Balancer

```go

package main

import (
	"container/heap"
	"fmt"
	"math/rand"
	"time"
)

const (
	NumWorkers  = 2
	NumItems    = 10
	NumRequests = 10
)

type Result struct {
	id, result int
}

type Request struct {
	id  int
	fn  func(int) int // The operation to perform.
	arg int
	c   chan Result // The channel to return the result.
}

type Worker struct {
	requests chan Request // work to do (buffered channel)
	pending  int          // count of pending tasks
	index    int          // index in the heap
}

func (w Worker) String() string {
	return fmt.Sprintf("Worker %d has %d requests pending",
		w.index, w.pending)
}

func (w *Worker) work(done chan<- *Worker) {
	go func() {
		for {
			req := <-w.requests                      // get Request from balancer
			req.c <- Result{req.id, req.fn(req.arg)} // call fn and send result
			done <- w                                // we've finished this request
		}
	}()
}

type Pool []*Worker

func (p Pool) start(done chan<- *Worker) {
	for _, w := range p {
		w.work(done)
	}
}
func (p Pool) Len() int {
	return len(p)
}

func (p Pool) Less(i, j int) bool {
	return p[i].pending < p[j].pending
}

func (p Pool) Swap(i, j int) {
	p[i], p[j] = p[j], p[i]
	p[i].index = i
	p[j].index = j

}

func (p *Pool) Pop() any {
	old := *p
	n := len(old)
	item := old[n-1]
	old[n-1] = nil  // don't stop the GC from reclaiming the item eventually
	item.index = -1 // for safety
	*p = old[0 : n-1]
	return item
}

func (p *Pool) Push(w any) {
	n := len(*p)
	item := w.(*Worker)
	item.index = n
	*p = append(*p, item)
}

type Balancer struct {
	pool Pool
	done chan *Worker
}

func (b *Balancer) startPool() {
	b.pool.start(b.done)
}

// Send Request to worker
func (b *Balancer) dispatch(req Request) {
	// Grab the least loaded worker...
	w := heap.Pop(&b.pool).(*Worker)
	// ...send it the task.
	w.requests <- req
	// One more in its work queue.
	w.pending++
	// Put it into its place on the heap.
	heap.Push(&b.pool, w)
}

// Job is complete; update heap
func (b *Balancer) completed(w *Worker) {
	// One fewer in the queue.
	w.pending--
	// Remove it from heap.
	heap.Remove(&b.pool, w.index)
	// Put it into its place on the heap.
	heap.Push(&b.pool, w)
}

func (b *Balancer) balance(work chan Request) {
	go func() {
		for {
			select {
			case req := <-work: // received a Request...
				b.dispatch(req) // ...so send it to a Worker
			case w := <-b.done: // a worker has finished ...
				b.completed(w) // ...so update its info
			}
		}
	}()
}

func main() {
	var pool Pool
	for i := 0; i < NumWorkers; i++ {
		pool.Push(newWorker())
	}

	req := make(chan Request)
	balancer := Balancer{pool: pool, done: make(chan *Worker)}
	balancer.startPool()
	balancer.balance(req)

	resp := sendLotsOfWorks(req)
	receiveLotsOfResults(resp)
}

func newWorker() *Worker {
	return &Worker{requests: make(chan Request, NumRequests)}
}

func newWork(in chan Request, id, arg int, fn func(int) int) <-chan Result {
	c := make(chan Result)
	in <- Request{id, fn, arg, c}
	return c
}

func sendLotsOfWorks(req chan Request) <-chan Result {
	fn := func(x int) int {
		time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
		return x * x
	}

	if NumItems < 2 {
		return newWork(req, 0, 100, fn)
	}

	resps := make([]<-chan Result, NumItems)
	for i := 0; i < NumItems; i++ {
		resps = append(resps, newWork(req, i, 100+i, fn))
	}

	return fanIn(resp...)
}

func receiveLotsOfResults(resp <-chan Result) {
	for i := 0; i < NumItems; i++ {
		fmt.Println(<-resp)
	}
}


func fanIn(c1 <-chan Result, c2 <-chan Result, rest ...<-chan Result) <-chan Result {
	c := fanIn2(c1, c2)
	for _, d := range rest {
		c = fanIn2(c, d)
	}
	return c
}

func fanIn2(c1 <-chan Result, c2 <-chan Result) <-chan Result {
	c := make(chan Result)
	go func() {
		for {
			select {
			case s := <-c1:
				c <- s
			case s := <-c2:
				c <- s
			}
		}
	}()
	return c
}

```
