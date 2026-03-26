---
name: dotnet-install
description: >
  Build, install, list, and remove .NET tools using dotnet-install.
  Use when the user wants to install, manage, or run .NET tools.
  Covers NuGet packages, GitHub repos, git URLs, and local projects.
---

# dotnet-install

## When to Use

- The user wants to install a .NET tool to their PATH
- The user wants to list, update, remove, or search for .NET tools
- The user wants to run a .NET tool without installing it (npx-style)
- The user needs help setting up dotnet-install itself

## When Not to Use

- The user is managing NuGet package references in a project (use `dotnet add package`)
- The user wants the traditional `dotnet tool install -g` workflow
- The user is working with .NET SDKs or runtimes (use `dotnet-install.sh` from Microsoft)

## Check if dotnet-install is available

```bash
dotnet-install --version
```

If not installed, install it:

```bash
# Option 1: via dotnet tool (requires .NET 10+ SDK)
dotnet tool install -g dotnet-install
dotnet-install doctor --fix

# Option 2: install script (Unix only, no SDK required)
curl -sSfL https://github.com/richlander/dotnet-install/raw/refs/heads/main/install.sh | sh
```

If already installed, update to the latest version:

```bash
dotnet-install update dotnet-install
```

## Install sources

Each source requires an explicit flag. With no arguments,
`dotnet-install` in a directory with a project builds and
installs it (like `dotnet publish`). With nothing to act on,
it prints help.

### From local source (builds)

Requires .NET SDK.

```bash
dotnet-install                      # current directory (like dotnet publish)
dotnet-install src/my-tool          # positional path
dotnet-install --project src/my-tool # explicit (like dotnet run --project)
dotnet-install app.cs               # file-based app (.NET 10+)
```

`--path` is an alias for `--project`.

### From NuGet (pre-built, no SDK required)

```bash
dotnet-install --package dotnetsay
dotnet-install --package dotnet-counters@9.0.0
```

### From GitHub (clones and builds)

Requires git and .NET SDK.

```bash
dotnet-install --github owner/repo            # tracks default branch, updatable
dotnet-install --github owner/repo --branch main   # tracks branch, updatable
dotnet-install --github owner/repo --tag v2.0      # pinned, no updates
dotnet-install --github owner/repo --rev abc123    # pinned, no updates
dotnet-install --github owner/repo@v2.0            # shorthand, pinned
dotnet-install --github owner/repo --ssh           # clone via SSH
```

### From any git URL (clones and builds)

Requires git and .NET SDK.

```bash
dotnet-install --git https://example.com/repo.git
dotnet-install --git https://example.com/repo.git --tag v1.0
```

When combined with `--github` or `--git`, `--project`
specifies a sub-path within the cloned repository.

## Git ref options

| Flag         | Pinned | Example                         |
|--------------|--------|---------------------------------|
| (none)       | no     | default branch, tracks upstream |
| `--branch`   | no     | named branch, tracks upstream   |
| `--tag`      | yes    | fixed tag, no updates           |
| `--rev`      | yes    | fixed commit SHA, no updates    |
| `@ref`       | yes    | shorthand in `--github` spec    |

Pinned installs are skipped by `dotnet-install update`.
To change versions, uninstall and reinstall.

## Subcommands

```bash
dotnet-install ls                  # list installed tools
dotnet-install rm <tool>           # remove a tool
dotnet-install update [tool]       # update one or all tools
dotnet-install search <query>      # search NuGet for tool packages
dotnet-install info <tool>         # show tool details and provenance
dotnet-install outdated            # check for newer versions
dotnet-install run <pkg> [args]    # run without installing (npx-style)
dotnet-install doctor [--fix]      # diagnose and fix PATH/config
dotnet-install env                 # print environment info
dotnet-install config [key] [val]  # view/set configuration
dotnet-install completion <shell>  # shell completion setup
```

## Install directory

Tools are installed to `~/.dotnet/bin/` by default.

| Override | Effect |
|----------|--------|
| `-o <dir>` | Custom output directory |
| `--local-bin` | Use `~/.local/bin/` instead |
| `DOTNET_TOOL_BIN` env var | Persistent override |

## PATH configuration

`dotnet-install` uses a dedicated env file (`~/.dotnet/bin/env`)
sourced from the shell rc file, following the same pattern as
Rustup (`~/.cargo/env`). Run `dotnet-install doctor --fix` to
configure PATH automatically. To activate in the current shell:

```bash
. "$HOME/.dotnet/bin/env"            # sh/bash/zsh
source "$HOME/.dotnet/bin/env.fish"  # fish
```

## Configuration

```bash
dotnet-install config                         # show all settings
dotnet-install config tip.quiet true          # suppress PATH tips
dotnet-install config manage-global-tools true  # drain dotnet global tools via doctor
```

## Reliable behavior

- Git updates verify that remote history is a
  continuation of local history. If a force push
  is detected, the update is refused — the user must
  uninstall and reinstall the tool.
- Pinned installs (`--tag`, `--rev`, `@ref`) are
  immutable. `update` skips them and reports the
  pinned ref. Changing versions requires an explicit
  uninstall and reinstall.
- Building from source requires the .NET SDK;
  `--package` works without the SDK.
- NuGet installs auto-enable roll-forward if the
  tool targets an older runtime.
- `--require-sourcelink` enforces SourceLink metadata
  in installed assemblies.
- Missing prereqs (git, .NET SDK) produce actionable
  error messages with install links.
