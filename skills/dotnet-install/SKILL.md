---
name: dotnet-install
description: >
  Build, install, list, and remove .NET tools using dotnet-install.
---

# dotnet-install

Install .NET executables to PATH — like `cargo install`
and `go install`. Build from source, install from NuGet,
or clone from GitHub.

## Install sources

Each source requires an explicit flag. With no arguments,
`dotnet-install` in a directory with a project builds and
installs it (like `dotnet publish`). With nothing to act on,
it prints help.

```bash
# Local project (default — works like dotnet publish)
dotnet-install                                # current directory
dotnet-install src/my-tool                    # positional path
dotnet-install --project src/my-tool          # explicit (like dotnet run --project)
dotnet-install app.cs                         # file-based app

# NuGet package (no SDK required)
dotnet-install --package dotnetsay
dotnet-install --package dotnet-counters@9.0.0  # pinned version

# GitHub repository
dotnet-install --github owner/repo            # tracks default branch, updatable
dotnet-install --github owner/repo --branch main   # tracks branch, updatable
dotnet-install --github owner/repo --tag v2.0      # pinned, no updates
dotnet-install --github owner/repo --rev abc123    # pinned, no updates
dotnet-install --github owner/repo@v2.0            # shorthand, pinned
dotnet-install --github owner/repo --ssh           # clone via SSH

# Any git URL
dotnet-install --git https://example.com/repo.git
dotnet-install --git https://example.com/repo.git --tag v1.0
```

`--path` is an alias for `--project`. When combined with
`--github` or `--git`, `--project` specifies a sub-path
within the cloned repository.

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
dotnet-install ls                # list installed tools
dotnet-install rm <tool>         # remove a tool
dotnet-install update [tool]     # update one or all tools
dotnet-install search <query>    # search NuGet
dotnet-install info <tool>       # show tool details
dotnet-install outdated          # check for newer versions
dotnet-install run <pkg> [args]  # run without installing
dotnet-install doctor            # diagnose PATH and config
dotnet-install env               # print environment info
dotnet-install completion <sh>   # shell completion setup
```

## Install directory

Tools are installed to `~/.dotnet/bin/` by default.
Override with `DOTNET_TOOL_BIN` env var, `-o <dir>`,
or `--local-bin` (uses `~/.local/bin/`).

## PATH configuration

`dotnet-install` uses a dedicated env file (`~/.dotnet/bin/env`)
that is sourced from the shell's rc file. Run
`dotnet-install doctor --fix` to configure PATH automatically.
To activate in the current shell without restarting:

```bash
. "$HOME/.dotnet/bin/env"          # sh/bash/zsh
source "$HOME/.dotnet/bin/env.fish" # fish
```

## Reliable behavior

- Git updates verify that the remote history is a
  continuation of the local history. If a force push
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
