# Skills

Custom skills for Claude Code.

## Installation

### Local Development

Test this plugin during development:

```bash
claude --plugin-dir /path/to/skills
```

### Install from Local Path

Add as a marketplace and install:

```bash
# Add as marketplace
/plugin marketplace add /path/to/skills

# Install the plugin
/plugin install skills@skills
```

### Install from GitHub

Once pushed to GitHub:

```bash
# Add marketplace
/plugin marketplace add owner/skills

# Install
/plugin install skills@skills
```

## Available Skills

| Skill | Description | Usage |
|-------|-------------|-------|
| `/skills:dotnet-inspect` | Inspect .NET assemblies and NuGet packages | `/dotnet-inspect` |

## Creating New Skills

1. Create a new directory under `skills/`:

```bash
mkdir skills/my-skill
```

2. Create `SKILL.md` with frontmatter and instructions:

```yaml
---
name: my-skill
description: When Claude should use this skill
argument-hint: [optional-args]
allowed-tools: Read, Grep, Bash
---

## My Skill

Instructions for Claude to follow...
```

### Skill Frontmatter Options

| Field | Description |
|-------|-------------|
| `name` | Display name (defaults to directory name) |
| `description` | When Claude should invoke this skill |
| `argument-hint` | Autocomplete hint for arguments |
| `allowed-tools` | Restrict which tools Claude can use |
| `disable-model-invocation` | Set `true` for manual-only skills |
| `user-invocable` | Set `false` for background knowledge |
| `context: fork` | Run in isolated subagent |
| `agent` | Subagent type: `Explore`, `Plan`, `general-purpose` |

### Dynamic Content

Use these placeholders in your skill instructions:

- `$ARGUMENTS` - All arguments passed to the skill
- `$0`, `$1`, `$2` - Individual arguments by position
- `` !`command` `` - Execute command and inject output

## Directory Structure

```text
skills/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/
│   └── dotnet-inspect/
│       └── SKILL.md
└── README.md
```

## License

MIT
