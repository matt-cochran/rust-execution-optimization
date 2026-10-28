# Optimization experiment record

```markdown
# <experiment id>: <performance contract>

## Workload
- revision / toolchain / target / features / profile:
- hardware / OS / limits / power policy:
- input distribution / payload / concurrency / warm-cold state:
- command or harness:
- correctness oracle:

## Contract
- primary metric and target:
- guardrails:
- noise/materiality threshold:
- rollback:

## Baseline
- raw artifact paths:
- samples and distribution (p50/p95/p99 where applicable):
- CPU / RSS / allocations / errors / throughput:
- profile and attributed bottleneck:

## Hypothesis
Metric X is limited by Y, evidenced by Z; change C should improve X by T without regressing G.

## Candidate
- bounded change:
- raw artifact paths:
- samples and distribution:
- correctness result:
- profile explanation:

## Decision
- delta with interval/noise interpretation:
- guardrail results:
- scope where the result is valid:
- adopt / reject / inconclusive:
- regression gate and owner:
```
