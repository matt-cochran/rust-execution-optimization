# Sources and design notes

Primary references:

- Rust Performance Book: <https://nnethercote.github.io/perf-book/>
- Rust Performance Book, benchmarking: <https://nnethercote.github.io/perf-book/benchmarking.html>
- Rust Performance Book, profiling: <https://nnethercote.github.io/perf-book/profiling.html>
- Criterion.rs book: <https://bheisler.github.io/criterion.rs/book/>
- cargo-flamegraph: <https://github.com/flamegraph-rs/flamegraph>
- Tokio tracing and tokio-console: <https://tokio.rs/tokio/topics/tracing-next-steps>
- rustc profile-guided optimization: <https://doc.rust-lang.org/rustc/profile-guided-optimization.html>
- rustc code-generation options: <https://doc.rust-lang.org/rustc/codegen-options/>

Design input:

- Boshen's Rust Performance Improvement Plan:
  <https://github.com/Boshen/rust-performance-improvement-plan>

The upstream plan was inspected at commit `e24edbafd997e92a0eaeb9013b0cf8c1c10ce985` on
2026-10-27. It has no declared license, so this skill does not reproduce its prose or examples.
It independently retains the general measure-profile-change-verify structure and improves it with:

- an explicit workload/SLO contract and independent correctness oracle;
- distributions, tail latency, noise thresholds and raw before/after evidence;
- off-CPU, async, contention, database and network diagnosis;
- a hierarchy that favors eliminating work before micro-optimization;
- conditional rather than blanket advice for hashers, allocators, unsafe code, inlining, LTO and
  CPU-specific code generation;
- production-shaped saturation checks, rollback and regression budgets;
- a CPM-compatible experiment artifact and integration benchmark.

Tool availability and platform support change. Verify current documentation and repository policy
before installing a profiler, enabling unstable Tokio instrumentation, or changing a release
profile. Never infer a generic speedup from another project.
