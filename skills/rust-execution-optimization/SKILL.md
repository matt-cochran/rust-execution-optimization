---
name: rust-execution-optimization
description: Use when a Rust program, service, worker, CLI, or library must improve runtime latency, throughput, CPU, memory, allocation rate, I/O, contention, or capacity. Builds a representative workload, profiles before editing, chooses one bounded experiment, preserves correctness, and accepts only a statistically credible before/after improvement with production-safe rollback. Not for compile, link, or test feedback time; use rust-cycle-time for those.
---

# Rust execution optimization

Optimize an observed workload, not Rust in the abstract. Read
[references/sources.md](references/sources.md) for the source basis and
[references/experiment-record.md](references/experiment-record.md) for the evidence artifact.

## 1. Turn the request into a performance contract

Name one primary objective: latency, throughput, CPU per operation, peak/steady RSS, allocated
bytes/allocations per operation, I/O, or capacity at an SLO. Define:

- the production-shaped input distribution, payload sizes, concurrency, data/cache state, and
  warm/cold mode;
- the primary metric and non-regression guardrails, especially p95/p99 latency, errors, memory and
  output correctness;
- target hardware, OS, Rust toolchain, revision, features and release profile;
- an independent correctness oracle and a rollback.

If no representative workload or correctness oracle exists, create that first. Never optimize a
toy input and generalize the result to production.

## 2. Establish a reproducible baseline

Use the production build and pin the environment. Record background load, CPU frequency/power
policy, affinity/container limits, allocator and external service versions. Warm up appropriately,
then collect enough repeated samples to report a distribution, not one stopwatch number.

For microbenchmarks, prefer Criterion with `black_box`, throughput units, saved baselines and a
declared noise threshold. For end-to-end services, report achieved throughput plus p50/p95/p99,
errors, CPU and RSS at several load points including saturation. Alternate baseline/candidate order
when machine drift is plausible. Preserve raw outputs.

## 3. Profile the limiting resource before editing

Start broad and select the tool that can explain the metric:

| Signal | Evidence |
|---|---|
| on-CPU time | `perf`, samply, Instruments, VTune, or `cargo flamegraph`; hardware counters when cache/branch behavior matters |
| off-CPU or I/O wait | syscall counts/traces, blocked-time or span traces, queue depth, storage/network/DB telemetry |
| allocations or memory | DHAT/dhat-rs, heaptrack, allocation counts/bytes, peak and steady RSS, representative cache occupancy |
| async stalls | tracing plus tokio-console task/resource views; separate busy, scheduled and idle time |
| lock/channel contention | wait/hold duration, queue depth, blocked tasks, critical-section profile |
| database/network boundary | query plans/counts, round trips, bytes, pool wait, downstream latency and retries |
| hot isolated function | Criterion plus instruction/cache counts (for example iai-callgrind) after macro profiling identifies it |

A flame graph chooses where to investigate; it does not prove the improvement. Include off-CPU and
external-boundary evidence when CPU samples do not explain wall time.

## 4. Form one falsifiable hypothesis

State: “Metric X is limited by Y, evidenced by Z; change C should improve X by at least T without
regressing guardrails G.” Choose the smallest correct experiment. Prefer this order:

1. eliminate unnecessary work or choose a better algorithm;
2. reduce round trips, syscalls, parsing, copies or repeated serialization;
3. improve data layout, locality, batching and reuse;
4. reduce demonstrated allocation pressure;
5. remove demonstrated lock/channel/task contention;
6. tune I/O and async concurrency with bounded backpressure;
7. use specialized search/SIMD/hash/allocator choices only for a measured hot path;
8. try LTO, PGO or code-layout optimization only after the workload is stable and representative.

Keep one causal change per experiment. A larger architectural change needs intermediate benchmarks
at stable seams so a regression can be attributed.

## 5. Reject folklore until evidence earns it

Do not apply these globally:

- replacing `HashMap` or its hasher without key-distribution, collision/HashDoS and workload proof;
- adding `#[inline(always)]`, `#[cold]`, unsafe unchecked indexing or `assert_unchecked` by intuition;
- replacing `Vec`, `String`, locks, channels or the allocator because an alternative is marketed as
  faster;
- adding parallelism without measuring saturation, coordination cost, queueing and tail latency;
- using `target-cpu=native` for portable artifacts;
- enabling fat LTO, one codegen unit, PGO or aggressive optimization without measuring runtime gain
  against build time, binary size and operational compatibility.

Unsafe optimizations need an invariant proof, focused tests, Miri/sanitizer coverage where
applicable, and a safe rollback. Prefer safe code when the gain is within noise.

## 6. Verify the candidate

Run identical baseline and candidate workloads on the same artifact class and environment. Require:

- all correctness tests/oracles pass, including malformed and boundary inputs;
- the confidence interval or repeated-run evidence clears the declared noise/materiality threshold;
- primary improvement holds across representative input classes, not only the easiest case;
- p95/p99, errors, CPU, RSS, allocations, binary size and downstream load meet guardrails;
- cold start, warm steady-state and saturation behavior are checked when relevant;
- a profile explains why the measured gain occurred and shows no bottleneck merely moved into an
  unmeasured dependency.

An inconclusive or noisy result is not a win. Rework the benchmark or reject the change.

## 7. Productionize and prevent regression

Store the benchmark/workload, raw before/after artifacts, profile, correctness results, decision,
scope and rollback using the experiment record. Add the narrowest stable regression budget to CI;
reserve noisy end-to-end capacity tests for controlled runners. Validate with a bounded canary or
production observation only when authorized, comparing the same workload/SLO metrics and retaining
an automatic rollback condition.

For a delivery plan, make the workload and baseline predecessor artifacts, each optimization a
bounded experiment, and the accepted benchmark/rollback receipt its completion evidence. Parallelize
independent hot paths only after the baseline identifies them; converge through one representative
system benchmark so local wins cannot hide an end-to-end regression.
