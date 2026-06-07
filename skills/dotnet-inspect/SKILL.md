---
name: dotnet-inspect
version: 0.9.6
description: Find evidence instead of guessing for .NET packages, platform libraries, local assemblies, APIs, dependencies, SourceLink/symbol provenance, and version-to-version API changes.
---

# dotnet-inspect

Use dotnet-inspect when you need evidence instead of guesses for .NET packages, platform libraries, local assemblies, APIs, dependencies, SourceLink/symbol provenance, or version-to-version API changes.

Invoke with `dnx`:

```bash
dnx dotnet-inspect -y -- <command>
```

For the authoritative workflow guide for the installed tool version, run:

```bash
dnx dotnet-inspect -y -- skill
```

Prefer that embedded skill output over this marketplace bootstrap file when commands or section names differ. It is versioned with the tool and prevents stale skill guidance after breaking CLI changes.

Default output is Markdown. Use `--oneline` for compact tabular output like `docker images`, and `--json` for structured automation. For output, query, and limiter details such as `-D`, bare `-S`, `-S "Async*"`, `-n`, `--tail`, and `--rows`, run the embedded guide.

## Fast starts

| Goal | Command |
| ---- | ------- |
| Find where a type lives | `find Pattern --oneline` |
| Inspect APIs | `type Type --package Foo`, then `member Type --package Foo` |
| Compare versions | `diff --package Foo@old..new --breaking` |
| Inspect package/library signals | `library Foo -S Signals` or `package Foo -S Signals` |
| Resolve SourceLink URLs | `source Type --package Foo --oneline` |
| Explore relationships | `depends Type`, `extensions Type`, `implements Interface` |

## Common workflow

Start broad, then carry resolved context forward:

```bash
dnx dotnet-inspect -y -- find JsonSerializer --oneline
dnx dotnet-inspect -y -- member JsonSerializer --package System.Text.Json
dnx dotnet-inspect -y -- source JsonSerializer --package System.Text.Json --oneline
```

Bare names use the router: platform-looking names are tried as installed platform libraries first, then fall back to NuGet packages if platform resolution fails. Use explicit `--platform <LibraryName>`, `--package Foo[@version]`, or `--library` when the source matters.

## Output control

Discover sections with `-D`, use bare `-S` for a curated high-density view, and select named sections with `-S Section`. Project table columns with `--columns`, fields with `--fields`, and count rows with `--count` when exactly one section is selected.

```bash
dnx dotnet-inspect -y -- member JsonSerializer --package System.Text.Json -D
dnx dotnet-inspect -y -- member JsonSerializer --package System.Text.Json -S Methods --columns "Name;Signature;Obsolete"
dnx dotnet-inspect -y -- library System.Text.Json -S "Async*" --count
```

## Guardrails

- First run `dnx dotnet-inspect -y -- skill` when doing non-trivial work; it contains the current, tool-embedded guidance.
- Quote generic type names and use `<T>`, not `<>`: `'Option<T>'`, `'INumber<TSelf>'`.
- `type` uses `-t` for type filters; `member` uses `-m` for member filters.
- Dotted member syntax works: `-m JsonSerializer.Deserialize`.
- Diff ranges use `..`: `--package Foo@1.0.0..2.0.0`.
- Use `--all` for non-public, hidden, and extra members; obsolete members are already shown by default.
