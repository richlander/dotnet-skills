---
name: dotnet-inspect
version: 0.10.5
description: Find evidence for .NET packages, platform libraries, assemblies, APIs, dependencies, SourceLink/source, and API version diffs.
---

# dotnet-inspect

Use dotnet-inspect when you need evidence instead of guesses for .NET packages, platform libraries, local assemblies, APIs, dependencies, SourceLink/source, or version-to-version API changes.

Invoke with `dnx`:

```bash
dnx dotnet-inspect -y -- <command>
```

This marketplace skill is intentionally only a bootstrapper. For non-trivial work, first run the version-matched embedded guide:

```bash
dnx dotnet-inspect -y -- skill
```

Prefer that embedded guide when commands, output modes, section names, or workflow guidance differ.

## Seed commands

| Goal | Command |
| ---- | ------- |
| Find where an API lives | `find Pattern` |
| Inspect types or members | `type Type --package Foo`, then `member Type --package Foo` |
| Compare versions | `diff --package Foo@old..new --breaking` |
| Inspect package/library signals | `package Foo -S Signals` or `library Foo -S Signals` |
| Locate source or implementation | `source Type --package Foo` or `member Type Member:1 -S "Decompiled Source"` |
| Explore relationships | `depends Type`, `extensions Type`, `implements Interface` |

After `find`, reuse the package, library, or platform scope it reports. Quote generic type names such as `'List<T>'`; use `<T>`, not `<>`.
