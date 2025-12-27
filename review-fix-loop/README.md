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
3. **Phase 0**: Subagent creates a TodoWrite checklist of ALL files to review
4. **Phase 1-5**: Review-fix cycles with 100% coverage requirement:
   - Read ALL files (parallel batches of 5-10)
   - Review for bugs, security, style, performance
   - Fix all issues found
   - Verify build passes
   - Repeat until clean AND 100% coverage achieved
5. Returns a structured summary with coverage report

## Why a Subagent?

- **Context isolation**: Review chatter stays out of main conversation
- **Fresh perspective**: Each pass has clean context
- **Thoroughness**: Opus model for deep analysis
- **Efficiency**: Only the summary consumes main context tokens

## v1.1.0 Changes

- **100% coverage requirement**: Agent must read every file before declaring "clean"
- **Phase 0 setup**: Creates TodoWrite checklist of all files upfront
- **Coverage report**: Final summary includes files reviewed X/Y (Z%)
- **Test file inclusion**: Explicitly requires reviewing test files
- **Parallel reading**: Reads files in batches for efficiency
