# Table Of Contents

# I. Language Basics

- [Constants]()

  - [Basic Constants](./basics/constant.md)

  - [Enumerated Constants](./basics/enum.md)

- [nil values](./basics/nil_values.md)

- [Functions]()

  - [Builtin Functions](./basics/builtin_functions.md)

  - [Closures](./basics/closure.md)

  - [Defer](./basics/defer.md)

# II. Data Structures

- [Arrays]()

  - [Basics](./datastruct/array.md)

- [Slices]()

  - [Basics](./datastruct/slices/basics.md)

  - [Tricks](./datastruct/slices/tricks.md)

  - [2D Slices](./datastruct/slices/2dslices.md)

- [Maps]()

  - [Basics](./datastruct/maps/basics.md)

- [Range Clause](./userdefined/range.md)

# III. Userdefined Types

- [Struct]()

  - [Basics](./userdefined/struct/basics.md)

- [Pointers](./userdefined/pointers.md)

- [Methods]()

  - [Basics](./userdefined/methods/basics.md)

  - [Addressability](./userdefined/methods/addressable.md)
  
  - [Receivers](./userdefined/methods/receivers.md)

  - [Method Values](./userdefined/methods/methodvalue.md)

  - [Method Expressions](./userdefined/methods/methodexp.md)

- [Interfaces]()

  - [Basics](./userdefined/interfaces/basics.md)

  - [Interface Values](./userdefined/interfaces/values.md)

  - [Empty Interfaces](./userdefined/interfaces/empty.md)

  - [Type Switch](./userdefined/interfaces/type_switch.md)

- [Composition]()

  - [Introduction](./userdefined/composition/introduction.md)

  - [Struct Embedding](./userdefined/composition/struct_embed.md)

  - [Interface Embedding](./userdefined/composition/interface_embed.md)

# IV. Errors

- [Introduction](./errors/intro.md)

- [Package errors](./errors/package.md)

# V. Generics

- [Basics]()

# VI. Standard Types And Utility Functions

- [IO Interfaces]()

  - [Reader]()

  - [Writer]()

  - [MultiReader]()

  - [MultiWriter]()

  - [TeeReader]()

  - [LimitReader]()

# VII. Concurrency

- [Basics]()

  - [Introduction](./concurrency/basics.md)

  - [Goroutines](./concurrency/goroutines.md)

  - [Channels](./concurrency/channels.md)

  - [Range And Close](./concurrency/range_and_close.md)

  - [Once And Others](./concurrency/once.md)

  - [WaitGroup](./concurrency/waitgroup.md)

- [Basic Patterns]()

  - [Signalling](./concurrency/patterns/singalling.md)

  - [for/select Idiom](./concurrency/patterns/for_select.md)

  - [Terminate Workers](./concurrency/patterns/terminate_workers.md)

  - [Verify Terminate](./concurrency/patterns/verify_termination.md)

  - [Closed Channels](./concurrency/patterns/closed_channels.md)

  - [Generator](./concurrency/patterns/generators.md)

  - [Multiplexrs](./concurrency/patterns/multiplexers.md)

  - [Sequencers](./concurrency/patterns/sequencers.md)

  - [Select Statement](./concurrency/patterns/select.md)

  - [Multiplexer using select](./concurrency/patterns/multiplexer_using_select.md)

  - [Timeout Message Dealy](./concurrency/patterns/timeout_message_delay.md)

  - [Timeout Conversation](./concurrency/patterns/timeout_conversation.md)

  - [Heartbeat](./concurrency/patterns/heartbeat.md)

  - [Quit Channel](./concurrency/patterns/quit_channel.md)

  - [Daisy Chain](./concurrency/patterns/daisy_chain.md)
  
  - [Reference](./concurrency/patterns/reference.md)

- [Advanced Concurrency]()

  - [Ownership Model](./concurrency/advanced/ownership.md)

  - [Nil Channel Trick](./concurrency/advanced/nil_channel.md)

  - [Buffered Channel Trick](./concurrency/advanced/buffered_chan.md)

  - [Unique ID](./concurrency/advanced/unique_id.md)

  - [Memory Recycler](./concurrency/advanced/mem_recycler.md)

  - [Pipelines](./concurrency/advanced/pipeline.md)

  - [Bounded Concurrency](./concurrency/advanced/bounded.md)

  - [Context](./concurrency/advanced/context.md)

  - [Package errgroup](./concurrency/advanced/errgroup.md)

  - [Package singleflight](./concurrency/advanced/singleflight.md)

  - [Developer Support](./concurrency/advanced/develper_support.md)

  - [Reference](./concurrency/advanced/reference.md)

- [Concurrency Examples]()

  - [Load Balancer](./concurrency/examples/load_balancer.md)

  - [Google Search](./concurrency/examples/fake_google_search.md)

  - [MD5 Digester]()

    - [Serial Digester](./concurrency/examples/digester/serial.md)

    - [Parallel Digester](./concurrency/examples/digester/parallel.md)

    - [Bounded Digester](./concurrency/examples/digester/bounded.md)

  - [Feed Reader]()

    - [A Naive Feed Reader]()
  
      - [Introduction](./concurrency/examples/feed_reader/naive_feed_reader.md)

      - [The Fetcher](./concurrency/examples/feed_reader/naive/fetcher.md)

      - [The Subscription](./concurrency/examples/feed_reader/naive/subscription.md)

      - [The Subscribe](./concurrency/examples/feed_reader/naive/subscribe.md)

      - [The Merge](./concurrency/examples/feed_reader/naive/merge.md)

      - [Code Walk](./concurrency/examples/feed_reader/naive/naive_implementation.md)

    - [List Of Bugs]()

      - [Introduction](./concurrency/examples/feed_reader/bugs_in_naivesub.md)

      - [Data Races](./concurrency/examples/feed_reader/bugs/data_race.md)

      - [Resource Leak](./concurrency/examples/feed_reader/bugs/resource_leak.md)

      - [Indefinite Blocking](./concurrency/examples/feed_reader/bugs/indefinite_blocking.md)

    - [A Good Feed Reader]()

      - [Introduction](./concurrency/examples/feed_reader/good_feed_reader.md)

      - [Fixing The Races](./concurrency/examples/feed_reader/good/data_races.md)

      - [Fixing The Leak](./concurrency/examples/feed_reader/good/resource_leak.md)

      - [Fixing The Blocking](./concurrency/examples/feed_reader/good/blocking.md)

      - [Code Walk](./concurrency/examples/feed_reader/good/good_implementation.md)

    - [Improving Feed Reader]()

      - [Introduction](./concurrency/examples/feed_reader/improved_reader.md)

      - [Filter Out Duplicates](./concurrency/examples/feed_reader/improvements/dedupe.md)

      - [Limit Pending Queue](./concurrency/examples/feed_reader/improvements/queue_size.md)

      - [Avoid Fetch Blocking](./concurrency/examples/feed_reader/improvements/avoid_blocking.md)

      - [Code Walk](./concurrency/examples/feed_reader/improvements/improved.md)

# VIII. Testing

- [Unit Tests]()

  - [Introduction](./testing/unit/intro.md)

  - [Table Driven](./testing/unit/tabledriven.md)

- [Test Coverage]()

  - [Introduction](./testing/cover/intro.md)

- [Benchmark Tests]()

  - [Introduction](./testing/benchmark/intro.md)

  - [Table Driven](./testing/benchmark/tabledriven.md)

# IX. Profiling

- [Basics]()
