# csharp-repomap

[![PyPI version](https://badge.fury.io/py/csharp-repomap.svg)](https://badge.fury.io/py/csharp-repomap)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**English** | **[简体中文](README.zh-CN.md)** | **[日本語](README.ja.md)**

> **Boost AI Agent efficiency for C# codebases** - Save tokens, improve accuracy, accelerate development.

## The Problem

AI coding agents (Claude Code, Cursor, Copilot) struggle with large C# codebases:

| Challenge | Impact |
|-----------|--------|
| **Context limit** | Can't see 1000+ files at once |
| **Blind spots** | Misses important classes, makes wrong assumptions |
| **Token waste** | Loads irrelevant code, burns context window |
| **Slow iteration** | Multiple rounds to understand structure |

## The Solution

**csharp-repomap** generates intelligent code maps that give AI agents a **bird's-eye view** of your codebase:

```
1000+ C# files  →  3 markdown files (~6k tokens total)
                   ├── L1: Module skeleton (what exists)
                   ├── L2: Class signatures (what matters)
                   └── L3: Reference graph (how they connect)
```

### Results

| Metric | Without RepoMap | With RepoMap |
|--------|-----------------|--------------|
| **Tokens per task** | 50k-100k | 10k-30k |
| **Code accuracy** | ~70% | ~95% |
| **Iterations needed** | 3-5 rounds | 1-2 rounds |
| **"File not found" errors** | Frequent | Rare |

## How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                    Your C# Codebase                         │
│                    (1000+ files)                            │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    csharp-repomap                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ Tree-sitter │→ │  Symbol     │→ │  PageRank   │         │
│  │ C# Parser   │  │  Extraction │  │  Ranking    │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
        ┌─────────┐   ┌─────────┐   ┌─────────┐
        │ L1 ~1k  │   │ L2 ~2k  │   │ L3 ~3k  │
        │ tokens  │   │ tokens  │   │ tokens  │
        └─────────┘   └─────────┘   └─────────┘
              │             │             │
              └─────────────┼─────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      AI Agent                               │
│  "I can see the entire codebase structure in 6k tokens!"   │
│  "I know which classes are important!"                      │
│  "I understand how modules connect!"                        │
└─────────────────────────────────────────────────────────────┘
```

## Key Features

| Feature | Benefit |
|---------|---------|
| **PageRank ranking** | AI sees important classes first, not random files |
| **Token-limited output** | Fits in context window, no overflow |
| **Layered detail** | L1 for overview → L2/L3 for deep dive |
| **Git hooks** | Auto-update on pull/merge, always fresh |
| **Cross-platform** | Windows, macOS, Linux notifications |

## Installation

```bash
pip install csharp-repomap
```

## Quick Start

```bash
# Initialize (choose your project type)
cd your-csharp-project
repomap init --preset unity    # Unity projects
repomap init --preset generic  # Other C# projects

# Generate the map
repomap generate --verbose

# Auto-update on git operations
repomap hooks --install
```

## Output Structure

Generated in `.repomap/output/`:

### L1 - Skeleton (~1k tokens)
```markdown
# MyProject Repo Map (L1)
> 45 modules | 320 classes | Generated: 2026-01-13

## Module Overview
- Player/ (12 classes) - Player management
- Combat/ (28 classes) - Battle system
- UI/ (45 classes) - User interface

## Core Entry Points
| Class | Module | Why Important |
|-------|--------|---------------|
| GameManager | Core | Central coordinator |
| PlayerService | Player | Player state management |
```

### L2 - Signatures (~2k tokens)
```markdown
# MyProject Repo Map (L2)

## GameManager (rank: 0.95)
+ Initialize() : void
+ Update(deltaTime: float) : void
+ GetService<T>() : T

## PlayerService (rank: 0.87)
+ LoadPlayer(id: string) : async Task<Player>
+ SavePlayer(player: Player) : async Task
```

### L3 - Relations (~3k tokens)
```markdown
# MyProject Repo Map (L3)

GameManager (refs: 15)
├── → PlayerService (uses)
├── → CombatSystem (uses)
├── → UIManager (uses)
└── ← SceneLoader (called by)
```

## Usage with AI Agents

### Claude Code
```bash
# Add to your CLAUDE.md or project context:
"Before implementing any feature, read .repomap/output/ to understand the codebase structure."
```

### Cursor / Copilot
Add `.repomap/output/` to your project's AI context or include in prompts.

### Example Prompt
> "Look at the L1 repo map to understand the module structure.
> Then check L2 for the PlayerService signatures.
> Now implement a new method to handle player inventory."

## Configuration

Edit `.repomap/config.yaml`:

```yaml
project_name: "My Game"

source:
  root_path: "Assets/Scripts"
  exclude_patterns:
    - "**/Editor/**"
    - "**/Tests/**"

# Token budgets per layer
tokens:
  l1_skeleton: 1000
  l2_signatures: 2000
  l3_relations: 3000

# Boost important class patterns
importance_boost:
  patterns:
    - prefix: "S"           # SPlayerService → boost
      boost: 2.0
    - suffix: "Manager"     # GameManager → boost
      boost: 1.5
```

## Presets

### Unity Preset
- Path: `Assets/Scripts`
- Boosts: `SXxx` service classes
- Categories: Core, Game, UI, Data, Network, Audio

### Generic Preset
- Path: `src`
- Boosts: `Service`, `Repository`, `Controller`
- Categories: Core, Domain, Application, API, Data

## Why PageRank?

Not all classes are equal. PageRank identifies **actually important** classes by analyzing the reference graph:

```
High PageRank (important):
  - Referenced by many other classes
  - Central to the architecture
  - AI should know about these first

Low PageRank (peripheral):
  - Utility classes, DTOs
  - Can be discovered on-demand
  - Don't waste tokens on these
```

## Requirements

- Python 3.8+
- Git (for hooks and commit info)
- Windows 10+ / macOS / Linux

## Contributing

Contributions welcome! Please submit a Pull Request.

## License

MIT License - see [LICENSE](LICENSE)

## Author

Created by [Yoji](https://github.com/sputnicyoji)

---

**Star this repo if it helps your AI coding workflow!**
