# Review-Fix Loop Plugin

Iterative code review and fix plugin for Claude Code. Spawns a subagent to review code until clean (max 5 passes), keeping the main conversation context clean.

## Installation

Since this is a private repo, use `settings.local.json` (gitignored) with your platform's absolute path.

### Linux / WSL / Devcontainer

Add to `.claude/settings.local.json`:

```json
{
  "enabledPlugins": {
    "review-fix-loop@local-plugins": true
  },
  "extraKnownMarketplaces": {
    "local-plugins": {
      "source": {
        "source": "directory",
        "path": "/workspaces/alpha-signals.02/plugins"
      }
    }
  }
}
```

### Windows

Add to `.claude/settings.local.json`:

```json
{
  "enabledPlugins": {
    "review-fix-loop@local-plugins": true
  },
  "extraKnownMarketplaces": {
    "local-plugins": {
      "source": {
        "source": "directory",
        "path": "C:\\path\\to\\alpha-signals\\plugins"
      }
    }
  }
}
```

### Quick Test (any platform)

```bash
claude --plugin-dir ./plugins/review-fix-loop
```

## Usage

The skill is **model-invoked** - Claude automatically uses it when you ask for iterative reviews:

```
"Review and fix src/Features/Auth/ until it's clean"
"Do a multi-pass review of the database layer"
"Comprehensively review and fix this module"
```

## How It Works

1. Claude recognizes the request matches the skill's triggers
2. Spawns a single subagent (Opus model) to do the work
3. Subagent performs up to 5 review-fix cycles:
   - Review for bugs, security, style, performance
   - Fix all issues found
   - Verify build passes
   - Repeat until clean or max passes reached
4. Returns a structured summary to the main context

## Why a Subagent?

- **Context isolation**: Review chatter stays out of main conversation
- **Fresh perspective**: Each pass has clean context
- **Thoroughness**: Opus model for deep analysis
- **Efficiency**: Only the summary consumes main context tokens
