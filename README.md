# LMAX Disruptor written in Go

The [Disruptor](https://lmax-exchange.github.io/disruptor/) was originally a library written in Java that provided a concurrent ring buffer data structure of the same name, developed at [LMAX Exchange](https://www.lmax.com/).

This repo is _yet-another_ port of the disruptor in Go. It is performant and free of heap allocation when running.

If for some reason you have Go code that needs to process messages at sub-microsecond latency, where shaving every nanosecond counts, then consider the disruptor pattern. Example situations:

* Financial trading systems (high-frequency trading)
* Real-time game servers (authoritative server logic)
* High-performance network packet processing (within a user-space application)
* Real-time data analytics/stream processing (very low latency pipelines)

## Key Considerations When Choosing a Disruptor Over Channels

* **Benchmark in your specific scenario**: Don't assume a disruptor is always better. Benchmark your application with both channels and your disruptor implementation to see if the latency reduction is actually significant and justifies the added complexity.
* **Complexity**: The disruptor is generally more complex to understand than Go channels. Make sure the performance gain outweighs the added complexity in development and maintenance.
* **Memory Management**: Disruptors often rely on pre-allocated buffers and ring buffer structures. Understand the memory implications and ensure you manage memory effectively, especially in long-running applications.
* **Garbage Collection**: While you are using Go, be mindful that even with a disruptor, GC can still run and introduce pauses.

## Comparisons to other Go ports

There _is_ already an existing port ([`smarty-prototypes/go-disruptor`](https://github.com/smarty-prototypes/go-disruptor)), but the key advantages of this library over that are:

* **Better encapsulation**: The user does not need to create and interact with the ring buffer directly (unless using `WriteBatch` and/or `BatchReadFunc`).
* **Batching support**: `WriteBatch` and `BatchReadFunc` allow the user to efficiently batch items in/out of the ring buffer, e.g. via SIMD code.
* **Generics support**: This library takes advantage of Go generics to simplify using the disruptor.

Nonetheless, these advantages come with slightly higher latency, on the order of `O(1 ns)`. Take this into consideration when deciding which LMAX port to use.

## Benchmarks

Benchmarks of 128-byte message throughput for `smarty-prototypes/go-disruptor`, `five-vee/disruptor`, and buffered Go channels. The producer and consumer run in their own goroutine. The buffer size is `1 << 22`.

_(Ran on my Macbook Air M3.)_

```zsh
$ go test -benchmem -run=^$ -bench . github.com/five-vee/go-disruptor/benchmarks
goos: darwin
goarch: arm64
pkg: github.com/five-vee/go-disruptor/benchmarks
cpu: Apple M3
BenchmarkDisruptor_22-8         112661708               10.26 ns/op            0 B/op          0 allocs/op
BenchmarkSmartystreets_22-8     131429239                9.208 ns/op           0 B/op          0 allocs/op
BenchmarkChannel_22-8           43865726                31.71 ns/op            0 B/op          0 allocs/op
PASS
ok      github.com/five-vee/go-disruptor/benchmarks     6.219s
```

## Features

- [x] Support single producer and single consumer.
- [ ] ~~Support multiple producers.~~ [^1]
- [x] Support multiple consumers.
- [x] Support different waiting strategies.
- [x] Support modifying the buffer directly.
- [x] Support consumer dependencies.
- [x] go.pkg.dev documentation.
- [x] Support producer and consumer batching.

[^1]: At the moment, multiple producers is explicitly not supported due to follow the [single writer principle](https://mechanical-sympathy.blogspot.com/2011/09/single-writer-principle.html). I.e. a single writer can write messages faster than multiple writers.
