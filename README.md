# Low-Latency Trading Engine

A software prototype of a low-latency order-processing pipeline, built to explore the concurrency and systems-engineering techniques real trading infrastructure relies on: multi-process architecture, shared-memory IPC, lock-free/wait-free synchronization, and percentile-based latency measurement.

> **Status:** 🚧 Phase 1 (V1) in active development. This is a learning/portfolio prototype, not production trading software — see [Scope](#scope) below.

## What This Is

A simulated order-processing pipeline — market data → strategy → risk/order management → matching — implemented as four separate OS processes communicating over shared memory, modeled on how real exchange and trading-firm systems are structured.

```
Market Data  →  Strategy  →  Risk + Order Management  →  Matching
   (process)      (process)         (process)              (process)
        \_____________ shared-memory IPC ______________/
```

- **Market Data** — simulates incoming price/quote data
- **Strategy** — generates trading signals and orders from that data
- **Risk + Order Management** — validates orders against pre-trade limits before they reach the book
- **Matching** — maintains the order book and matches orders on a price-time priority basis

## Key Engineering Techniques

- Multi-process architecture with shared-memory IPC (not sockets, to stay off the syscall path on the hot path)
- Spinlock and wait-free-where-practical synchronization for shared structures
- Double-buffered snapshots for lock-free, torn-write-free reads of the order book
- Cache-line-aware data layout to avoid false sharing on hot-path structures
- Latency measured as percentiles (P50 / P99 / P99.9 / max), not just averages, in line with how real systems are monitored

## Tech Stack

| Layer | Language | Purpose |
|---|---|---|
| Core engine | C++ | Order book, matching, risk engine, order management, latency instrumentation |
| Tooling | Python | Benchmarking, test-data generation, correctness checks, reporting |

## Scope

**In scope for V1:** a small, fixed instrument set; in-process simulation of market data and order flow; price-time matching; basic risk checks (pre-trade limits, kill switch, fat-finger guards); latency/throughput benchmarking.

**Out of scope for V1:** real network I/O or exchange connectivity, persistence/crash recovery, colocation or kernel bypass (DPDK/RDMA), arbitrary multi-asset support.

**The honest ceiling:** this prototype adopts the *software-side* concurrency techniques real trading systems use. It does not and cannot close the *hardware* gap — colocation, kernel bypass, FPGA feed parsing — that separates a fast software prototype from a real production system. That gap is physical, not something more software optimization fixes.

## Design Documentation

The full engineering spec — locked design decisions, build order, and a non-negotiables checklist — lives in [`docs/DESIGN.md`](docs/DESIGN.md).

## Build & Run

Coming soon — instructions will be added once the Phase 1 pipeline is functional end-to-end.

## License

TBD