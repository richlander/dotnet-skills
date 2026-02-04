# dotnet-skills

.NET skills for coding assistants. The skills describe how to use specific .NET tools that accelerate LLM-driven .NET + C# development.

## Getting Started

Use `/plugin` to manage marketplaces and install plugins. For a complete walkthrough with screenshots for both Copilot and Claude Code, see [Skill marketplace walkthrough](https://github.com/richlander/dotnet-skills/issues/1).

**Copilot users**: Start with `/plugin marketplace add richlander/dotnet-skills`

## Available Skills

| Skill | Description |
|-------|-------------|
| `dotnet-inspect` | Inspect .NET assemblies and NuGet packages |

## Installation

```bash
# Add marketplace
/plugin marketplace add richlander/dotnet-skills

# Install plugin
/plugin install dotnet-skills@richlander-dotnet-skills
```

## Updating

To get the latest version of the plugin, first update the marketplace to refresh available versions, then update the installed plugin:

```bash
# Update marketplace to fetch latest versions
/plugin marketplace update richlander-dotnet-skills

# Update the installed plugin
claude plugin update dotnet-skills@richlander-dotnet-skills
```

You can also update the marketplace through the interactive UI by running `/plugin`, navigating to the **Marketplaces** tab, selecting `richlander-dotnet-skills`, and choosing **Update marketplace**.

## Documentation

### Copilot

- [About Agent Skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)
- [Using Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli)
- [copilot-plugins](https://github.com/github/copilot-plugins) - Official plugins repository

### Claude Code

- [Plugins](https://docs.anthropic.com/en/docs/claude-code/plugins)
- [Agent Skills](https://docs.anthropic.com/en/docs/claude-code/skills)
