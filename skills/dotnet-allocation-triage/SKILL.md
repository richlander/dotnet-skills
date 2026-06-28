---
name: dotnet-allocation-triage
version: 0.1.0
description: Evidence-based static allocation triage of compiled .NET assemblies with dotnet-inspect, paired with profiler/benchmark confirmation. Use to find heap-allocation hot spots (boxing, closures, per-call arrays, in-loop LINQ/string building) before reaching for a profiler.
---

# .NET Allocation Triage

Find real, fixable heap allocations in compiled .NET code, then confirm the win with a benchmark or profiler before changing anything.

`dotnet-inspect`'s `Performance Triage` reads the actual IL of a built assembly (and its dependencies), so it sees allocations a source-text scan misses and produces a receipt — the exact member, IL offset, and decompiled source — for every finding. It is the cheap static first pass; a profiler or benchmark is the confirmation.

Invoke with `dnx` (no install needed):

```bash
dnx dotnet-inspect -y -- library <path-to.dll> -S "Performance Triage"
```

## When to Use

- Triaging allocation hot spots in a built assembly or its dependencies
- Turning a vague "it allocates too much" into specific, located, evidence-backed findings
- A fast first pass before (or to focus) an allocation profiler or BenchmarkDotNet run

## When Not to Use

- Code that is not on a hot path — these are micro-optimizations; do not apply them to startup, config, or one-time init
- Proving an improvement — static triage finds candidates; only a benchmark or profiler proves a regression or a win

## Workflow

### Build, then triage the binary

Triage runs on compiled IL, so build first (Release), then point at the DLL:

```bash
dotnet build -c Release
dnx dotnet-inspect -y -- library bin/Release/<tfm>/<Assembly>.dll -S "Performance Triage"
```

The section is already ordered to surface pay-dirt first — in-loop allocations, then by confidence, then by call-graph reach. So for a quick top-findings view, cap the rows and you get the highest-value sites directly, no filtering needed:

```bash
dnx dotnet-inspect -y -- library bin/Release/<tfm>/<Assembly>.dll -S "Performance Triage" -n 15 --rows
```

For machine-readable output add `--json` (or `--tsv`/`--jsonl`) — useful when you want every row to group or filter by shape yourself.

### Read the table

Each row is one allocation site:

| Column | Meaning |
| ------ | ------- |
| Member | The method that allocates |
| Root Reach | How reachable the member is from the public API surface — a prioritization signal, higher is hotter |
| Shape | The allocation pattern (see below) |
| Evidence | What was found in the IL |
| Fix | The suggested change, including any caveat |
| Confidence | `high` or `medium` — `medium` carries a caveat to check |
| Loop | `loop` when the allocation is on a loop back-edge (repeated every iteration) |
| IL | The IL offset of the site |

Prioritize: `loop` + `high` confidence + higher `Root Reach` first. Always read the `Fix` text — it states when the fix only reduces (not eliminates) the allocation, or when it depends on escape/version.

### Shapes

| Shape | What it is | Typical fix / caveat |
| ----- | ---------- | -------------------- |
| `box-value-type` | A value type boxed onto the heap | Use a generic API, value-typed overload, or interpolation. Only flagged when the box escapes |
| `capturing-delegate` | A closure over captured state | A `static` local function with explicit state parameters. On .NET 10+ the JIT can stack-allocate a non-escaping closure, so the win is largest when it escapes or runs in a loop |
| `instance-method-group-delegate` | A delegate that binds a receiver | Cache it in a field when the receiver is stable, or use a static method with explicit state |
| `string-build-in-loop` | `s += …` accumulation in a loop (O(n^2) copies) | Build with a `StringBuilder` hoisted out of the loop, `ToString()` once after |
| `linq-scan-in-loop` | A LINQ membership/search terminal re-scanning a sequence each iteration | Precompute a set/dictionary index once outside the loop. Quadratic only if the scanned sequence grows with the loop |
| `scan-method-in-loop-call` | A method that scans a sequence, called inside a loop | Same fix as above, across the call boundary |
| `small-array` | `new[]` of a small constant length | A span or `stackalloc` may avoid it when the array does not escape |
| `span-to-array-copy` / `temporary-byte-array-copy` | A span/value materialized into a throwaway array | Let the span flow to the consumer; use `BinaryPrimitives`/`stackalloc` for byte conversions |
| `allocation-hotspot` | A method allocating densely inside a loop with no single dominant shape | Reduce per-iteration allocations; pool or reuse buffers |

### Get the receipt

Before changing code, confirm the site by reading its decompiled source. Pass the type's simple name and the member name, with `--library` pointing at the same DLL:

```bash
dnx dotnet-inspect -y -- member '<Type>' <Method> --library <path-to.dll> -S "Decompiled Source"
```

For a member inside a package or platform library, use `--package <Pkg>` or `--platform <Lib>` instead of `--library`.

### Confirm with a profiler or benchmark

Static triage finds the candidate; a dynamic tool proves the change is worth it:

- Micro: wrap the hot method in a BenchmarkDotNet benchmark with `[MemoryDiagnoser]`; compare allocated bytes before/after.
- Running app: capture a GC/allocation trace with `dotnet-trace collect --profile gc-verbose`, or watch `System.Runtime` GC counters with `dotnet-counters monitor`, and check the member appears and shrinks after the fix.

Re-run the triage after the fix to confirm the row is gone.

## Pitfalls

- Do not blindly apply every row. Low `Root Reach` or non-`loop` sites are often not worth touching.
- `medium` confidence means the `Fix`/caveat must be checked — e.g. a `small-array` that escapes still needs the array; a `linq-scan-in-loop` over a constant-size sequence is fine.
- Intrinsic framework costs (e.g. Blazor render plumbing) are already suppressed; if a finding looks framework-mandated, verify it is genuinely your code's allocation.
- A cleared shape can still allocate — the `Fix` text notes when a rewrite only reduces the allocation (e.g. a closure removed but a LINQ iterator still allocated).

## Tool guide

This skill is a focused workflow. For the full command surface, output modes, and section names, run the version-matched embedded guide:

```bash
dnx dotnet-inspect -y -- skill
```

> ⚠️ Findings are heuristics over IL. They locate candidates with evidence; always confirm impact with a benchmark or profiler and human review before changing production code.
