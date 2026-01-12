# csharp-repomap

Generate layered code maps for C# projects, optimized for AI assistants like Claude.

## What is it?

**csharp-repomap** creates structured code maps that help AI assistants understand your C# codebase. It generates three levels of detail:

- **L1 Skeleton** (~1k tokens): Module overview and core entry classes
- **L2 Signatures** (~2k tokens): Class and method signatures
- **L3 Relations** (~3k tokens): Reference graph between classes

The tool uses **tree-sitter** for accurate C# parsing and **PageRank** algorithm to identify the most important classes in your codebase.

## Features

- **Tree-sitter parsing**: Accurate C# syntax analysis
- **PageRank ranking**: Identify important classes by reference count
- **Token-limited output**: Fits within AI context windows
- **Git hooks**: Auto-update on pull/merge/checkout
- **Windows notifications**: Toast notifications when updates complete
- **Unity preset**: Pre-configured for Unity projects

## Installation

```bash
pip install csharp-repomap
```

For token counting (optional):
```bash
pip install csharp-repomap[tiktoken]
```

## Quick Start

```bash
# Initialize in your project (choose preset)
cd your-csharp-project
repomap init --preset unity    # For Unity projects
repomap init --preset generic  # For other C# projects

# Generate repo map
repomap generate --verbose

# Check status
repomap status

# Install Git hooks for auto-update
repomap hooks --install
```

## Configuration

After `repomap init`, edit `.repomap/config.yaml`:

```yaml
project_name: "My Project"

source:
  root_path: "Assets/Scripts"  # Path to your C# source
  exclude_patterns:
    - "**/Editor/**"
    - "**/Tests/**"

tokens:
  l1_skeleton: 1000
  l2_signatures: 2000
  l3_relations: 3000

importance_boost:
  patterns:
    - prefix: "S"           # SPlayerService
      boost: 2.0
    - suffix: "Manager"     # GameManager
      boost: 1.5
```

## Output Files

Generated in `.repomap/output/`:

| File | Description |
|------|-------------|
| `repomap-L1-skeleton.md` | Module overview, categories, core classes |
| `repomap-L2-signatures.md` | Top classes with method signatures |
| `repomap-L3-relations.md` | Reference graph (who calls whom) |
| `repomap-meta.json` | Git info, statistics, timestamps |

## Git Hooks

Auto-update repo map when code changes:

```bash
# Install hooks
repomap hooks --install

# Uninstall hooks
repomap hooks --uninstall
```

Hooks trigger on:
- `git pull`
- `git merge`
- `git checkout` (branch switch)

## Usage with Claude Code

1. Add `.repomap/output/` to your project context
2. Claude will see the L1/L2/L3 files and understand your codebase structure
3. The maps auto-update as you pull new code

Example prompt:
> "Look at the repo map to understand the module structure, then implement..."

## Presets

### Unity Preset
- Configured for `Assets/Scripts`
- Boosts `SXxx` service classes
- Categories: Core, Game, UI, Data, Network, Audio

### Generic Preset
- Configured for `src` directory
- Boosts `Service`, `Repository`, `Controller` patterns
- Categories: Core, Domain, Application, API, Data

## Requirements

- Python 3.8+
- Git (for hooks and commit info)
- Windows 10+ (for Toast notifications)

## License

MIT License - see [LICENSE](LICENSE)
