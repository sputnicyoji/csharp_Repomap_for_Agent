# csharp-repomap

[![PyPI version](https://badge.fury.io/py/csharp-repomap.svg)](https://badge.fury.io/py/csharp-repomap)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**[English](README.md)** | **简体中文** | **[日本語](README.ja.md)**

> **提升 AI Agent 在 C# 代码库中的效率** - 节约 Token，提升准确率，加速开发。

## 问题背景

AI 编码 Agent（Claude Code、Cursor、Copilot）在处理大型 C# 代码库时面临挑战：

| 挑战 | 影响 |
|------|------|
| **上下文限制** | 无法同时查看 1000+ 文件 |
| **盲区** | 遗漏重要类，做出错误假设 |
| **Token 浪费** | 加载无关代码，消耗上下文窗口 |
| **迭代缓慢** | 需要多轮对话才能理解结构 |

## 解决方案

**csharp-repomap** 生成智能代码地图，为 AI Agent 提供代码库的**鸟瞰视角**：

```
1000+ C# 文件  →  3 个 Markdown 文件（总计约 6k tokens）
                   ├── L1: 模块骨架（有什么）
                   ├── L2: 类签名（什么重要）
                   └── L3: 引用关系图（如何连接）
```

### 效果对比

| 指标 | 无 RepoMap | 有 RepoMap |
|------|------------|------------|
| **每任务 Token 消耗** | 50k-100k | 10k-30k |
| **代码准确率** | ~70% | ~95% |
| **所需迭代轮数** | 3-5 轮 | 1-2 轮 |
| **"找不到文件"错误** | 频繁 | 罕见 |

## 工作原理

```
┌─────────────────────────────────────────────────────────────┐
│                    你的 C# 代码库                            │
│                    (1000+ 文件)                              │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    csharp-repomap                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ Tree-sitter │→ │   符号      │→ │  PageRank   │         │
│  │ C# 解析器   │  │   提取      │  │   排序      │         │
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
│  "我用 6k tokens 就能看到整个代码库结构！"                    │
│  "我知道哪些类是重要的！"                                    │
│  "我理解各模块如何连接！"                                    │
└─────────────────────────────────────────────────────────────┘
```

## 核心特性

| 特性 | 收益 |
|------|------|
| **PageRank 排序** | AI 优先看到重要类，而非随机文件 |
| **Token 限制输出** | 适配上下文窗口，不会溢出 |
| **分层细节** | L1 概览 → L2/L3 深入 |
| **Git hooks** | pull/merge 时自动更新，保持最新 |
| **跨平台** | Windows、macOS、Linux 通知支持 |

## 安装

```bash
pip install csharp-repomap
```

## 快速开始

```bash
# 初始化（选择项目类型）
cd your-csharp-project
repomap init --preset unity    # Unity 项目
repomap init --preset generic  # 其他 C# 项目

# 生成地图
repomap generate --verbose

# 设置 git 操作时自动更新
repomap hooks --install
```

## 输出结构

生成在 `.repomap/output/` 目录：

### L1 - 骨架（约 1k tokens）
```markdown
# MyProject 代码地图 (L1)
> 45 模块 | 320 类 | 生成时间: 2026-01-13

## 模块概览
- Player/ (12 个类) - 玩家管理
- Combat/ (28 个类) - 战斗系统
- UI/ (45 个类) - 用户界面

## 核心入口点
| 类名 | 模块 | 为何重要 |
|------|------|----------|
| GameManager | Core | 中央协调器 |
| PlayerService | Player | 玩家状态管理 |
```

### L2 - 签名（约 2k tokens）
```markdown
# MyProject 代码地图 (L2)

## GameManager (rank: 0.95)
+ Initialize() : void
+ Update(deltaTime: float) : void
+ GetService<T>() : T

## PlayerService (rank: 0.87)
+ LoadPlayer(id: string) : async Task<Player>
+ SavePlayer(player: Player) : async Task
```

### L3 - 关系（约 3k tokens）
```markdown
# MyProject 代码地图 (L3)

GameManager (refs: 15)
├── → PlayerService (使用)
├── → CombatSystem (使用)
├── → UIManager (使用)
└── ← SceneLoader (被调用)
```

## 配合 AI Agent 使用

### Claude Code
```bash
# 添加到 CLAUDE.md 或项目上下文：
"实现任何功能前，先阅读 .repomap/output/ 理解代码库结构。"
```

### Cursor / Copilot
将 `.repomap/output/` 添加到项目的 AI 上下文中。

### 示例提示词
> "查看 L1 代码地图理解模块结构。
> 然后检查 L2 获取 PlayerService 的签名。
> 现在实现一个处理玩家背包的新方法。"

## 配置

编辑 `.repomap/config.yaml`：

```yaml
project_name: "我的游戏"

source:
  root_path: "Assets/Scripts"
  exclude_patterns:
    - "**/Editor/**"
    - "**/Tests/**"

# 每层的 Token 预算
tokens:
  l1_skeleton: 1000
  l2_signatures: 2000
  l3_relations: 3000

# 提升重要类模式的权重
importance_boost:
  patterns:
    - prefix: "S"           # SPlayerService → 提升
      boost: 2.0
    - suffix: "Manager"     # GameManager → 提升
      boost: 1.5
```

## 预设配置

### Unity 预设
- 路径：`Assets/Scripts`
- 提升：`SXxx` 服务类
- 分类：Core、Game、UI、Data、Network、Audio

### 通用预设
- 路径：`src`
- 提升：`Service`、`Repository`、`Controller`
- 分类：Core、Domain、Application、API、Data

## 为什么用 PageRank？

不是所有类都同等重要。PageRank 通过分析引用图识别**真正重要**的类：

```
高 PageRank（重要）：
  - 被许多其他类引用
  - 架构中心位置
  - AI 应该优先了解

低 PageRank（边缘）：
  - 工具类、DTO
  - 可以按需发现
  - 不要在这些上浪费 Token
```

## 系统要求

- Python 3.8+
- Git（用于 hooks 和提交信息）
- Windows 10+ / macOS / Linux

## 贡献

欢迎贡献！请提交 Pull Request。

## 许可证

MIT 许可证 - 详见 [LICENSE](LICENSE)

## 作者

由 [Yoji](https://github.com/sputnicyoji) 创建

---

**如果这个工具对你的 AI 编码工作流有帮助，请 Star！**
