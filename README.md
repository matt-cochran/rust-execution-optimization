# rust-execution-optimization

Reusable agent skill for evidence-backed Rust runtime optimization. It turns a performance goal
into a representative workload, profiles the limiting resource, makes one bounded change, and
accepts it only when correctness and before/after evidence support the result.

Install `skills/rust-execution-optimization` with your agent's skill installer, or copy that
directory into the repository/user skill directory supported by a `SKILL.md`-compatible host.

The skill covers latency, throughput, memory, allocation, I/O, async scheduling, contention,
database/network boundaries, binary optimization, statistical gates, production validation, and
rollback. Compile/build/test feedback belongs in the separate
[`rust-cycle-optimization`](https://github.com/matt-cochran/rust-cycle-optimization) skill.
