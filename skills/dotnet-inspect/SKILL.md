---
name: dotnet-inspect
version: 0.26.0
description: Find evidence instead of guessing for .NET packages, platform libraries, assemblies, APIs, dependencies, source, performance, and version-to-version changes.
---

# dotnet-inspect

Use dotnet-inspect when you need evidence instead of guesses about compiled
.NET code. It inspects packages, platform libraries, local assemblies, restored
projects, APIs, dependencies, metadata, source provenance, and implementation.

Run it with `dnx`:

```bash
dnx dotnet-inspect -y -- <command>
```

`-y` accepts the tool package prompt; `--` passes the remaining arguments to
dotnet-inspect.

## Basic UX

- Start with `find Pattern` when you do not know where an API lives.
- Inspect roots with `package Foo`, `library Foo`, or `library path/to.dll`.
- Reuse the package, library, or platform scope reported by `find` with
  `type Type --package Foo` or `member Type Member:1 --package Foo`.
- Select `-S Source` on one exact Type or Member for authored-first source.
- Add `--project path/to/project` for restored project dependencies.
- Compare versions with `diff --package Foo@old..new --breaking`.
- Use `-D` to discover sections, `-S` to select them, and `-Q` to discover
  query facets and operators.
- Default output is Markdown; use `--table`, `--tsv`, `--jsonl`, or `--json`
  when another shape is more useful.
- Quote generic type names such as `'List<T>'`.

## Embedded guides

The tool ships version-matched guidance. Run `skill` for the router, `skill
list` for the inventory, or `skill <name>` for one focused guide.

| Skill | Use it for |
| ----- | ---------- |
| `query` | Discovery, selection, projection, output formats, and limits |
| `package-skills` | Skills provided by restored NuGet packages |
| `private-feeds` | Custom feeds, credentials, caches, and offline use |
| `compatibility` | API and implementation changes between versions |
| `correctness` | Exceptions, error handling, and unsafe operations |
| `signals` | Provenance, compatibility, dependency, and safety signals |
| `sourcelink` | PDB mappings, source locations, and verified source |
| `metadata` | Raw ECMA-335 tables, heaps, handles, and rows |
| `decompiler` | Decompiled C#, annotated source, IL, and fidelity |
| `performance` | Library metrics, call-graph leverage, and allocation/performance triage |
| `relationships` | Dependencies, package call graphs, callers, implementors, and extensions |

For non-trivial work, load the relevant embedded guide and prefer it whenever
commands, sections, output shapes, or workflow guidance differ from this
bootstrapper.
