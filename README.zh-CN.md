# csharp-repomap

[![PyPI version](https://badge.fury.io/py/csharp-repomap.svg)](https://badge.fury.io/py/csharp-repomap)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**[English](README.md)** | **简体中文** | **[日本語](README.ja.md)**

> 为 C# 项目生成分层代码地图，专为 Claude 等 AI 助手优化。

## 这是什么？

**csharp-repomap** 创建结构化的代码地图，帮助 AI 助手理解你的 C# 代码库。它生成三个层次的详细信息：

| 层级 | Token 数 | 内容 |
|------|----------|------|
| **L1 骨架** | ~1k | 模块概览、分类、核心入口类 |
| **L2 签名** | ~2k | 重要类及其方法签名 |
| **L3 关系** | ~3k | 引用关系图（谁调用谁） |

该工具使用 **tree-sitter** 进行精确的 C# 解析，并使用 **PageRank** 算法识别代码库中最重要的类。

## 为什么需要它？

AI 助手的上下文窗口有限。当处理大型代码库（1000+ 文件）时，它们无法看到全貌。**csharp-repomap** 通过以下方式解决这个问题：

1. **优先处理重要代码** - PageRank 识别核心类
2. **分层细节** - 从骨架开始，按需深入
3. **Token 意识** - 适应上下文限制
4. **自动更新** - Git hooks 保持地图最新

## 功能特性

- **Tree-sitter 解析** - 精确的 C# 语法分析
- **PageRank 排序** - 通过引用计数识别重要类
- **Token 限制输出** - 适应 AI 上下文窗口
- **Git hooks** - 在 pull/merge/checkout 时自动更新
- **跨平台通知** - Windows Toast、macOS、Linux
- **Unity 预设** - 为 Unity 项目预配置
- **通用预设** - 适用于任何 C# 项目

## 安装

```bash
pip install csharp-repomap
```

可选的 token 计数功能：
```bash
pip install csharp-repomap[tiktoken]
```

## 快速开始

```bash
# 在项目中初始化（选择预设）
cd your-csharp-project
repomap init --preset unity    # Unity 项目
repomap init --preset generic  # 其他 C# 项目

# 生成代码地图
repomap generate --verbose

# 查看状态
repomap status

# 安装 Git hooks 自动更新
repomap hooks --install
```

## 配置

运行 `repomap init` 后，编辑 `.repomap/config.yaml`：

```yaml
project_name: "我的项目"

source:
  root_path: "Assets/Scripts"  # C# 源码路径
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

## 输出文件

生成在 `.repomap/output/` 目录：

| 文件 | 描述 |
|------|------|
| `repomap-L1-skeleton.md` | 模块概览、分类、核心类 |
| `repomap-L2-signatures.md` | 重要类及方法签名 |
| `repomap-L3-relations.md` | 引用关系图 |
| `repomap-meta.json` | Git 信息、统计、时间戳 |

## Git Hooks

代码变更时自动更新地图：

```bash
# 安装 hooks
repomap hooks --install

# 卸载 hooks
repomap hooks --uninstall
```

触发时机：
- `git pull`
- `git merge`
- `git checkout`（切换分支）

## 配合 Claude Code 使用

1. 将 `.repomap/output/` 添加到项目上下文
2. Claude 将看到 L1/L2/L3 文件并理解代码库结构
3. 拉取新代码时地图自动更新

示例提示词：
> "查看 repo map 理解模块结构，然后实现..."

## 预设配置

### Unity 预设
- 配置为 `Assets/Scripts` 路径
- 提升 `SXxx` 服务类权重
- 分类：Core、Game、UI、Data、Network、Audio

### 通用预设
- 配置为 `src` 目录
- 提升 `Service`、`Repository`、`Controller` 模式权重
- 分类：Core、Domain、Application、API、Data

## 系统要求

- Python 3.8+
- Git（用于 hooks 和提交信息）
- Windows 10+ / macOS / Linux（用于通知）

## 贡献

欢迎贡献！请随时提交 Pull Request。

## 许可证

MIT 许可证 - 详见 [LICENSE](LICENSE)

## 作者

由 [Yoji](https://github.com/sputnicyoji) 创建

---

**如果觉得有用，请 Star 这个仓库！**
