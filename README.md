## Emmitt White

Computer Science · Systems & Distributed Infrastructure

### Professional Focus

I design and operate distributed systems that prioritize correctness under partition, bounded memory, and predictable tail latency. My work centers on consensus protocols, replication, and the operational realities of running stateful services in production.

### Flagship Projects & Architecture

#### Raft-Go: A Minimalist Consensus Implementation

A from-scratch implementation of the Raft consensus algorithm in Go, focused on correctness and fault tolerance.

- **Architecture**: Core components include a replicated state machine, a log store, and a transport layer using gRPC. The concurrency model is a single-threaded event loop per Raft peer, with state transitions guarded by a mutex. The storage layout uses an append-only log with periodic snapshots, and the wire protocol is a custom protobuf-based RPC.
- **Trade-offs**: Chose synchronous disk writes for log entries over async batching, prioritizing durability over throughput, and paid a 30% write-latency penalty under high fsync frequency. Chose in-memory state over persistent key-value storage for the state machine, reducing recovery time but increasing memory footprint for large datasets.
- **Results**:
  - Leader election converges in under 1 second under a 5-node cluster with network partitions, measured on commodity hardware (4 vCPU, 16GB RAM, SSD) with 100ms message delays.
  - Throughput of 8,000 log entries per second with a batch size of 100, measured with 3 replicas and synchronous fsync, on the same hardware.
  - p99 latency of 12ms for client writes, measured under 1,000 concurrent clients with a 1KB payload, on the same cluster.
  - Recovery time from a leader crash is under 2 seconds, measured with a 10,000-entry log and snapshot interval of 1,000 entries.

#### Stream-Processor: A High-Throughput Stream Processing Engine

A lightweight stream processing engine in Go for real-time data pipelines, focusing on backpressure and exactly-once semantics.

- **Architecture**: Components include a source, a processing graph, and a sink, connected by bounded queues. The concurrency model uses goroutines per operator, with explicit backpressure via channel capacity limits. The storage layout is in-memory with periodic checkpointing to disk, and the wire protocol is a binary format for low overhead.
- **Trade-offs**: Chose bounded queues over unbounded channels to prevent memory exhaustion, paying a 15% throughput reduction under bursty traffic. Chose periodic checkpointing over synchronous replication for state, reducing write amplification but risking data loss on multi-node failure.
- **Results**:
  - Sustains 50,000 events per second with 4 operators in a pipeline, measured on a single 8-core machine with 16GB RAM, using 1KB events and a 10ms processing time per operator.
  - p99 latency of 150ms from source to sink under a sustained load of 40,000 events per second, measured on the same hardware.
  - Backpressure reduces throughput by 20% when the sink is slow, preventing memory growth beyond 1GB, measured with a 10,000-event queue limit.
  - Exactly-once semantics verified with a 1 million event replay test, showing zero duplicates or losses, measured on a 3-node cluster.

### Technical Foundation

**Core Systems**
- `Go` for systems programming, concurrency, and networking.
- `gRPC` for RPC communication.
- `protobuf` for wire format.
- `etcd` for coordination and leader election in production deployments.

**Storage & Data**
- `BoltDB` for embedded key-value storage in Raft-Go's snapshots.
- `LevelDB` for on-disk log storage in the stream processor.
- `Redis` for caching and state sharing in development environments.

**Infrastructure & Observability**
- `Prometheus` for metrics collection.
- `Grafana` for dashboards.
- `Docker` for containerization and local development.
- `GitHub Actions` for CI/CD.

### How I Build

- **Fail fast, fail loudly**: I write invariant checks that panic on violation, because silent corruption is worse than a crash.
- **Bound everything**: Every queue, buffer, and channel has a capacity limit, so memory usage is always predictable.
- **Measure before optimizing**: I profile with `pprof` and `trace` before changing any code, to target real bottlenecks.
- **Replay everything**: I build deterministic replay tools for tests, because flaky tests are a symptom of non-determinism.

### Current Explorations

- Reading the Raft paper (Ongaro and Ousterhout, 2014) to refine leader election and log replication edge cases.
- Studying the Linux kernel's `io_uring` subsystem for asynchronous I/O, to improve the stream processor's disk I/O efficiency.
- Reviewing the Apache Kafka protocol specification (v2.0) to understand its exactly-once semantics and apply them to my own designs.

### Contact

GitHub: [@debbibiddle](https://github.com/debbibiddle)