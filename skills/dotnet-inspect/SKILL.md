---
name: dotnet-inspect
version: 0.10.5
description: Find evidence instead of guessing for .NET packages, platform libraries, local assemblies, APIs, dependencies, SourceLink/symbol provenance, and version-to-version API changes.
---

# dotnet-inspect

Use dotnet-inspect when you need evidence instead of guesses for .NET packages, platform libraries, local assemblies, APIs, dependencies, SourceLink/symbol provenance, or version-to-version API changes.

Invoke with `dnx`:

```bash
dnx dotnet-inspect -y -- <command>
```

This marketplace skill is intentionally only a bootstrapper. For the authoritative workflow guide for the installed tool version, run:

```bash
dnx dotnet-inspect -y -- skill
```

Prefer that embedded skill output over this file when commands, output modes, section names, or workflow guidance differ. It is versioned with the tool and prevents stale marketplace guidance after CLI changes.

## Fast starts

| Goal | Command |
| ---- | ------- |
| Find where a type lives | `find Pattern` |
| Inspect APIs | `type Type --package Foo`, then `member Type --package Foo` |
| Compare versions | `diff --package Foo@old..new --breaking` |
| Inspect package/library signals | `library Foo -S Signals` or `package Foo -S Signals` |
| Locate source or implementation | `source Type --package Foo` or `member Type Member:1 -S "Decompiled Source"` |
| Explore relationships | `depends Type`, `extensions Type`, `implements Interface` |

## Common workflow

Start with the embedded guide, then carry resolved context forward:

```bash
dnx dotnet-inspect -y -- skill
dnx dotnet-inspect -y -- find JsonSerializer
dnx dotnet-inspect -y -- member JsonSerializer --package System.Text.Json
dnx dotnet-inspect -y -- source JsonSerializer --package System.Text.Json
```

After `find`, reuse the package or library it reports in follow-up commands. Use explicit `--platform <LibraryName>`, `--package Foo[@version]`, or `--library` when the source matters.

## Output control

Default output is Markdown. Use `--table` for compact human scanning, `--tsv` for stable field splitting, `--jsonl` for one JSON object per table row, `--json` for structured object graphs, and `--mermaid` for graph-shaped output such as `depends`.

Discover sections and columns with `-D`; select sections with `-S Section`, wildcards such as `-S "Async*"`, or categories such as `-S @All`. Project table columns with `--columns`, fields with `--fields`, and count rows with `--count` when one table section is selected. Use `-n`, `--tail`, and `--rows` instead of shell pipes when limiting output.

```bash
dnx dotnet-inspect -y -- member JsonSerializer --package System.Text.Json -D --tsv
dnx dotnet-inspect -y -- member JsonSerializer --package System.Text.Json -S Methods --columns "Name;Signature;Obsolete"
dnx dotnet-inspect -y -- library System.Text.Json -S "Async*" --count
```

## Guardrails

- First run `dnx dotnet-inspect -y -- skill` when doing non-trivial work; it contains the current, tool-embedded guidance.
- Quote generic type names and use `<T>`, not `<>`: `'Option<T>'`, `'INumber<TSelf>'`.
- `type` uses `-t` for type filters; `member` uses `-m` for member filters.
- Dotted member syntax works: `-m JsonSerializer.Deserialize`.
- Diff ranges use `..`: `--package Foo@1.0.0..2.0.0`.
- Unpinned packages use the latest stable by default; add `--preview` when prerelease APIs matter.
- Use `--all` for non-public, hidden, and extra members; obsolete members are already shown by default.
