# csharp-repomap

[![PyPI version](https://badge.fury.io/py/csharp-repomap.svg)](https://badge.fury.io/py/csharp-repomap)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> Generate layered code maps for C# projects, optimized for AI assistants like Claude.

## What is it?

**csharp-repomap** creates structured code maps that help AI assistants understand your C# codebase. It generates three levels of detail:

| Level | Tokens | Content |
|-------|--------|---------|
| **L1 Skeleton** | ~1k | Module overview, categories, core entry classes |
| **L2 Signatures** | ~2k | Top classes with method signatures |
| **L3 Relations** | ~3k | Reference graph (who calls whom) |

The tool uses **tree-sitter** for accurate C# parsing and **PageRank** algorithm to identify the most important classes in your codebase.

## Why?

AI assistants have limited context windows. When working with large codebases (1000+ files), they can't see the full picture. **csharp-repomap** solves this by:

1. **Prioritizing important code** - PageRank identifies core classes
2. **Layered detail** - Start with skeleton, drill down as needed
3. **Token-conscious** - Fits within context limits
4. **Auto-updating** - Git hooks keep maps fresh

## Features

- **Tree-sitter parsing** - Accurate C# syntax analysis
- **PageRank ranking** - Identify important classes by reference count
- **Token-limited output** - Fits within AI context windows
- **Git hooks** - Auto-update on pull/merge/checkout
- **Cross-platform notifications** - Windows Toast, macOS, Linux
- **Unity preset** - Pre-configured for Unity projects
- **Generic preset** - Works with any C# project

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

### Example L1 Output

```markdown
# MyGame Repo Map (L1)
> Generated: 2026-01-12 | Commit: abc1234

## Module Overview (45 modules)

### Core Systems
- Player/ (12 classes) - Player management
- Combat/ (28 classes) - Battle system

### Top 10 Classes by Importance
| Rank | Class | Module | Score |
|------|-------|--------|-------|
| 1 | GameManager | Core | 0.95 |
| 2 | PlayerService | Player | 0.87 |
```

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

## How It Works

```
                    +-----------------+
                    |   C# Source     |
                    |   Files (.cs)   |
                    +-----------------+
                            |
                            v
                    +-----------------+
                    |   Tree-sitter   |
                    |   C# Parser     |
                    +-----------------+
                            |
                            v
                    +-----------------+
                    |   Symbol        |
                    |   Extraction    |
                    +-----------------+
                            |
                            v
                    +-----------------+
                    |   PageRank      |
                    |   Ranking       |
                    +-----------------+
                            |
                +-----------+-----------+
                |           |           |
                v           v           v
            +------+    +------+    +------+
            |  L1  |    |  L2  |    |  L3  |
            +------+    +------+    +------+
```

## Requirements

- Python 3.8+
- Git (for hooks and commit info)
- Windows 10+ / macOS / Linux (for notifications)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT License - see [LICENSE](LICENSE)

## Author

Created by [Yoji](https://github.com/sputnicyoji)

---

**Star this repo if you find it useful!**
