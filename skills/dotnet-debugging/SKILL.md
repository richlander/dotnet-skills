---
name: dotnet-debugging
version: 0.10.6
description: Use dotnet diagnostic tools plus dotnet-inspect to explain crash, dump, trace, and sparse IL-coordinate artifacts.
---

# dotnet-debugging

Use this skill when a .NET debugging or diagnostics task can be agentified with
`dotnet-*` tools and static explanation from `dotnet-inspect`.

This skill is about the workflow across tools. For exact `dotnet-inspect`
commands and section names, prefer the version-matched embedded guide:

```bash
dnx dotnet-inspect -y -- skill
```

## Core loop

1. **Collect a runtime artifact** with a dotnet diagnostic tool.
2. **Extract sparse coordinates**: method identity plus IL offset when available.
3. **Normalize coordinates** into a text file:

   ```text
   # label coordinate
   crash-frame 0x06000042+0x2F
   hot-sample 0x06000051+0x10
   ```

4. **Explain with static facts**:

   ```bash
   dnx dotnet-inspect -y -- library MyApp.dll --il-offsets coords.txt
   ```

5. **Drill in** when needed:

   ```bash
   dnx dotnet-inspect -y -- library MyApp.dll --il-offset 0x06000042+0x2F -S "Return Address Context"
   dnx dotnet-inspect -y -- member My.Type DoWork --library MyApp.dll -S "Allocation Facts"
   ```

## Crash dump workflow

Use when an app crashes, hangs, or throws in production-like conditions.

```bash
dotnet-dump collect --process-id <pid> --output crash.dmp
```

Then inspect the dump with SOS/debugger commands appropriate to the host:

```text
# Example shape; exact commands vary by host/tooling.
# stack / clrstack / dumpstack -> method names and IL/native offsets
# ip2md / name2ee / dumpmd -> MethodDesc/metadata identity
# dumpil / u / disassemble -> map instruction offsets
```

Agent task:

- preserve the raw dump-tool output,
- identify the module/assembly containing each interesting frame,
- convert method identity to a MethodDef token when possible,
- convert frame/native instruction positions to IL offsets when available,
- write `coords.txt`,
- run `dotnet-inspect --il-offsets`.

If token+IL cannot be recovered, report that explicitly and fall back to
member/type inspection by name.

## Trace/profiler workflow

Use when an EventPipe trace, profiler, or sampled profile points at hot methods
or sparse offsets.

```bash
dotnet-trace collect --process-id <pid> --output trace.nettrace
```

Agent task:

- use the producer's export/symbolication tools to find method names and offsets,
- normalize any MethodDef token + IL offset pairs to `coords.txt`,
- label rows with the producer signal (`hot-sample`, `alloc-sample`, `exception-sample`),
- run `dotnet-inspect --il-offsets`,
- use `Performance Triage` only after objective facts suggest a likely problem.

## Analyzer / CI artifact workflow

Use when a test harness, static analyzer, or custom CI tool emits IL locations.

Preferred artifact shape:

```text
assembly: MyApp.dll
coordinate: 0x06000042+0x2F
reason: suspicious allocation
```

Agent task:

- turn the artifact into `label coordinate` rows,
- keep malformed lines as comments or labels for traceability,
- run `dotnet-inspect --il-offsets`,
- quote the resulting `Meaning` and `Evidence` columns in issues or PRs.

## What to skillify next

Prefer adding concrete adapter guidance only after a scenario is proven useful:

- dump stack to coordinates,
- trace/profiler samples to coordinates,
- analyzer artifact to coordinates.

Do not duplicate `dotnet-inspect` command reference here. Link or defer to the
embedded `dotnet-inspect skill` output for current section names and flags.
