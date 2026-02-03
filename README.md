# dotnet-skills

.NET skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). The skills describe how use specific .NET tools that accelerate LLM-driven .NET + C# development.

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

- [Claude Code Plugins](https://docs.anthropic.com/en/docs/claude-code/plugins)
- [Agent Skills](https://docs.anthropic.com/en/docs/claude-code/skills)
