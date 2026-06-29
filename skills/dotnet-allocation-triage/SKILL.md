---
name: dotnet-allocation-triage
version: 0.1.0
description: Cross-tool workflow to reduce .NET heap allocations — static triage with dotnet-inspect, then confirm the win with BenchmarkDotNet or dotnet-trace/dotnet-counters before changing code. Use when chasing allocation hot spots on a hot path.
---

# .NET Allocation Triage Workflow

Find allocation hot spots statically, confirm they matter dynamically, fix, then verify — pairing `dotnet-inspect` (static, IL-precise) with a benchmark or profiler (dynamic, ground truth). This skill is the *workflow*; for tool capabilities, output modes, shapes, and flags, defer to the version-matched guide:

```bash
dnx dotnet-inspect -y -- skill
```

## Workflow

1. **Triage statically.** Build Release, then run `dotnet-inspect`'s `Performance Triage` over the assembly to get a ranked list of pay-dirt allocation sites (member, shape, fix, confidence, in-loop). The cheapest first pass — works on a binary and its dependencies, no profiler needed. See the embedded guide for the exact command and filters.
2. **Confirm it's real.** Static triage finds *candidates*; only a dynamic tool proves a win:
   - **Micro:** wrap the hot method in a BenchmarkDotNet benchmark with `[MemoryDiagnoser]`; record allocated bytes before/after.
   - **Running app:** capture a GC/allocation trace with `dotnet-trace collect --profile gc-verbose`, or watch `System.Runtime` GC counters with `dotnet-counters monitor`, and confirm the member shows up and shrinks.
3. **Fix**, then **re-triage** to confirm the row is gone and re-benchmark to confirm the allocation dropped.

## When to use / not use

- Use on hot-path code with a measured or suspected allocation problem.
- Don't micro-optimize cold paths (startup, config, one-time init).
- Don't ship a fix on static triage alone — confirm with a benchmark or profiler first.

> Findings are heuristics over IL: candidates with evidence, not proof. Always confirm impact with a benchmark or profiler and human review before changing production code.
