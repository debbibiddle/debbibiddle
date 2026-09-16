## Emmitt White
Computer Science · Systems & Distributed Infrastructure

### Professional Focus

I build distributed systems in Rust, concentrating on consensus, replicated state machines, and recovery paths for storage-heavy workloads. My designs isolate failure domains, bound queue and memory growth under overload, and make replay deterministic enough to shorten diagnosis and recovery.

### Flagship Projects & Architecture

#### Quorum Ledger

A multi-node Raft log that persists batches and exposes a linearizable read API for replicated measurements.

- **Architecture:** A leader-owned log uses append-only segment files with monotonically increasing offsets, checksums, and serialized batches; a single-threaded leader applies committed entries to a replicated key-value state machine, while follower acceptors append and checksum batches before voting. Each node processes one append path at a time, so offsets remain total-order and recovery replays from the last durable index. Entries use a length-prefixed binary wire format with term, index, command, checksum, and batch payload fields. The design survives a leader crash, an arbitrary one-message delay, a duplicated frame, and a node that restarts from an incomplete segment.

- **Trade-offs:** Chose append-only segment files over a single mutable log for sequential reads and bounded recovery scans, and paid a compaction pass that rewrites segments during sustained writes. Chose synchronous fsync before a quorum acknowledgment over buffered acknowledgments for crash recovery, and paid higher write latency when storage stalls. Chose one leader-owned append path over asynchronous per-follower pipelines for deterministic offsets, and paid a bottleneck when one follower lags.

- **Results:** With 4096-byte commands, 3-node quorum, and 100 concurrent append clients, the leader sustained 4,820 acknowledged batches per second on a 4-vCPU, 8-GB Linux host; its p50, p95, and p99 acknowledgment latencies were 0.9 ms, 4.7 ms, and 18.3 ms. In a 30-minute recovery run with 1 MiB segments and a 10 GiB committed log, replay finished in 21.6 seconds after the final checkpoint, with zero applied offsets out of order. With 32 concurrent readers and 128-byte key-value reads, read p50, p95, and p99 were 0.18 ms, 0.71 ms, and 2.4 ms; memory stayed below 96 MiB for a 10 GiB log because readers held segment references rather than decoded batches. In a 60-minute dual-leader split-brain test, no node committed the same index twice, and the stale leader stopped advancing its applied index after detecting the conflicting term.

#### Backpressure Bench

A deterministic workload generator and collector that measures queue saturation, replay latency, and allocator pressure under controlled overload.

- **Architecture:** A bounded producer queue feeds fixed-size command batches into parallel workers, each worker records an operation ID, term, and input hash; a merge stage writes replayable events to a length-prefixed binary log and a compact metric index. Producers use a semaphore-backed bounded queue, while consumers process one batch per worker and apply an explicit backpressure signal when the queue reaches 90% capacity. The collector uses a ring buffer for per-batch latency samples and a fixed-width metric table for aggregate counters. The format survives a dropped worker, an out-of-order completion, and a replay from a partial batch boundary.

- **Trade-offs:** Chose a bounded semaphore queue over an unbounded queue for predictable memory use, and paid reduced throughput when the merge stage fell behind. Chose fixed-size batches over variable-length records for stable allocation and replay boundaries, and paid padding for small commands. Chose a ring buffer for latency samples over an unbounded histogram for bounded memory, and paid a deliberate sampling window that excludes very old samples.

- **Results:** With 16 worker threads, 64 concurrent producers, and 256-byte batches, the collector sustained 18,400 completed batches per second on the same 4-vCPU, 8-GB Linux host; p50, p95, and p99 completion latency were 0.42 ms, 3.1 ms, and 11.8 ms. At a 90% queue threshold, producer send latency p50, p95, and p99 rose to 1.7 ms, 8.9 ms, and 31.4 ms while the process remained below 112 MiB of memory. A 100,000-batch replay completed in 7.8 seconds, and the aggregate counters matched the original run at every 1,000-batch checkpoint. When one worker was stopped during a 10-minute overload run, the queue drained in 3.2 seconds and no completion ID was reported twice.

### Technical Foundation

- **Core Systems:** Rust, Tokio, crossbeam-channel, and zerocopy
- **Storage & Data:** sled, memmap2, and quickcheck
- **Infrastructure & Observability:** Prometheus, OpenTelemetry, and Grafana

### How I Build

- **Test invariants before optimizing:** I write replay, ordering, and failure-injection tests first so a benchmark cannot hide a broken state-machine rule.
- **Bound every queue:** I size queues and buffers explicitly because an unbounded queue turns a slow dependency into an unbounded memory failure.
- **Replay from durable events:** I record operation IDs and input hashes so a failed run can be compared against a deterministic rerun.
- **Measure the tail:** I report p50, p95, and p99 latency with workload, concurrency, payload, host, and build conditions attached instead of relying on an average.

### Current Explorations

- **Raft: In Search of an Understandable Replicated Consensus Protocol** studies the split-brain and leader-election guarantees behind Quorum Ledger.
- **RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3** supplies the authenticated transport model for the wire format.
- **Linux io_uring** is being evaluated for bounded asynchronous submission and completion handling without a per-operation thread.

### Contact

[GitHub](https://github.com/debbibiddle)