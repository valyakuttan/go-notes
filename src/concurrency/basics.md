# Concurrency Basics

> Parallelism is doing lot of things simultaneously, but concurrency
> is dealing lot of things simultaneously

## Concurrency

- Concurrency is the composition of independently executing computations.

- Concurrency is a way to structure software, particularly as a way to
  write clean code that interacts well with the real world.

- Concurrency is not parallelism, although it enables parallelism.

- If you have only one processor, your program can still be concurrent but
  it cannot be parallel. On the other hand, a well-written concurrent
  program might run efficiently in parallel on a multiprocessor.

Concurrency is a model for software construction, which gives the ability
to easily understand, use and reason about the system.

Parallelism is not the goal of concurrency, its only goal is a better
structure. But concurrency may use parallelism to do a better job.
