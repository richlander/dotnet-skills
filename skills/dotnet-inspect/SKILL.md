---
name: dotnet-inspect
version: 0.8.1
description: Query .NET APIs across NuGet packages, platform libraries, and local files. Use for factual answers about package contents, API signatures, compatibility changes, relationships, SourceLink, and assembly metadata.
---

# dotnet-inspect

Use dotnet-inspect when you need evidence about .NET libraries instead of guessing: API signatures, package contents, extension methods, implementors, SourceLink URLs, dependencies, or version-to-version API changes.

Invoke through `dnx` unless installed:

```bash
dnx dotnet-inspect -y -- <command>
```

For the full version-specific workflow guide, run:

```bash
dnx dotnet-inspect -y -- skill
```

Default output is Markdown. Use `--oneline` to scan, `--json` for structured data, `--count` to count one selected table section, and `-n N` to limit output without losing headers.

## Fast starts

| Goal | Command |
| ---- | ------- |
| Find where a type lives | `find Pattern --oneline` |
| Inspect APIs | `type Type --package Foo`, then `member Type --package Foo` |
| Compare versions | `diff --package Foo@old..new --breaking` |
| Audit package/library | `package Foo`, `library Foo`, or `library ./path/to.dll` |
| Resolve SourceLink URLs | `source Type --package Foo --oneline` |
| Explore relationships | `depends Type`, `extensions Type`, `implements Interface` |

## Common workflow

Start broad, then carry resolved context forward:

```bash
dnx dotnet-inspect -y -- find JsonSerializer --oneline
dnx dotnet-inspect -y -- member JsonSerializer --package System.Text.Json
dnx dotnet-inspect -y -- source JsonSerializer --package System.Text.Json --oneline
```

Bare names use the router: platform-looking names are tried as installed platform libraries first, then fall back to NuGet packages if platform resolution fails. Use explicit `--platform <LibraryName>`, `--package Foo[@version]`, or `--library` when the source matters. For upgrades, start with `diff`, then inspect affected members:

```bash
dnx dotnet-inspect -y -- diff --package System.Text.Json@9.0.0..10.0.0 --breaking
dnx dotnet-inspect -y -- member JsonSerializer --package System.Text.Json@10.0.0
```

## Output control

Discover sections with `-D` or bare `-S`, then select with `-S Section`. Project table columns with `--columns`, fields with `--fields`, and count rows with `--count` when exactly one section is selected.

```bash
dnx dotnet-inspect -y -- member JsonSerializer --package System.Text.Json -D
dnx dotnet-inspect -y -- member JsonSerializer --package System.Text.Json -S Methods --columns "Name;Signature;Obsolete"
dnx dotnet-inspect -y -- library System.Text.Json -S "Async*" --count
```

## Guardrails

- Quote generic type names and use `<T>`, not `<>`: `'Option<T>'`, `'INumber<TSelf>'`.
- `type` uses `-t` for type filters; `member` uses `-m` for member filters.
- Dotted member syntax works: `-m JsonSerializer.Deserialize`.
- Diff ranges use `..`: `--package Foo@1.0.0..2.0.0`.
- Use `--all` for non-public, hidden, and extra members; obsolete members are already shown by default.
