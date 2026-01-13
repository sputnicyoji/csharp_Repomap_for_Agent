# csharp-repomap

[![PyPI version](https://badge.fury.io/py/csharp-repomap.svg)](https://badge.fury.io/py/csharp-repomap)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**[English](README.md)** | **[简体中文](README.zh-CN.md)** | **日本語**

> **C#コードベースでのAI Agentの効率を向上** - トークン節約、精度向上、開発加速。

## 課題

AIコーディングエージェント（Claude Code、Cursor、Copilot）は大規模なC#コードベースで苦戦します：

| 課題 | 影響 |
|------|------|
| **コンテキスト制限** | 1000+ファイルを同時に見られない |
| **盲点** | 重要なクラスを見落とし、誤った仮定をする |
| **トークン浪費** | 無関係なコードを読み込み、コンテキストを消費 |
| **遅い反復** | 構造理解に複数ラウンド必要 |

## 解決策

**csharp-repomap** はAI Agentにコードベースの**俯瞰図**を提供するインテリジェントなコードマップを生成：

```
1000+ C#ファイル  →  3つのMarkdownファイル（合計約6kトークン）
                     ├── L1: モジュールスケルトン（何があるか）
                     ├── L2: クラスシグネチャ（何が重要か）
                     └── L3: 参照グラフ（どう繋がるか）
```

### 効果比較

| 指標 | RepoMapなし | RepoMapあり |
|------|-------------|-------------|
| **タスクあたりトークン** | 50k-100k | 10k-30k |
| **コード精度** | ~70% | ~95% |
| **必要な反復回数** | 3-5ラウンド | 1-2ラウンド |
| **「ファイルが見つからない」エラー** | 頻繁 | まれ |

## 仕組み

```
┌─────────────────────────────────────────────────────────────┐
│                    あなたのC#コードベース                     │
│                    (1000+ファイル)                           │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    csharp-repomap                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ Tree-sitter │→ │  シンボル   │→ │  PageRank   │         │
│  │ C#パーサー  │  │   抽出      │  │  ランキング │         │
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
│  「6kトークンでコードベース全体の構造が見える！」              │
│  「どのクラスが重要か分かる！」                               │
│  「モジュール間の接続が理解できる！」                          │
└─────────────────────────────────────────────────────────────┘
```

## 主な機能

| 機能 | メリット |
|------|----------|
| **PageRankランキング** | ランダムなファイルではなく重要なクラスを優先表示 |
| **トークン制限出力** | コンテキストウィンドウに収まる |
| **階層化された詳細** | L1で概要 → L2/L3で詳細 |
| **Git hooks** | pull/merge時に自動更新、常に最新 |
| **クロスプラットフォーム** | Windows、macOS、Linux通知対応 |

## インストール

```bash
pip install csharp-repomap
```

## クイックスタート

```bash
# 初期化（プロジェクトタイプを選択）
cd your-csharp-project
repomap init --preset unity    # Unityプロジェクト
repomap init --preset generic  # その他のC#プロジェクト

# マップを生成
repomap generate --verbose

# git操作時の自動更新を設定
repomap hooks --install
```

## 出力構造

`.repomap/output/` ディレクトリに生成：

### L1 - スケルトン（約1kトークン）
```markdown
# MyProject コードマップ (L1)
> 45モジュール | 320クラス | 生成日時: 2026-01-13

## モジュール概要
- Player/ (12クラス) - プレイヤー管理
- Combat/ (28クラス) - 戦闘システム
- UI/ (45クラス) - ユーザーインターフェース

## コアエントリポイント
| クラス | モジュール | 重要な理由 |
|--------|----------|------------|
| GameManager | Core | 中央コーディネーター |
| PlayerService | Player | プレイヤー状態管理 |
```

### L2 - シグネチャ（約2kトークン）
```markdown
# MyProject コードマップ (L2)

## GameManager (rank: 0.95)
+ Initialize() : void
+ Update(deltaTime: float) : void
+ GetService<T>() : T

## PlayerService (rank: 0.87)
+ LoadPlayer(id: string) : async Task<Player>
+ SavePlayer(player: Player) : async Task
```

### L3 - リレーション（約3kトークン）
```markdown
# MyProject コードマップ (L3)

GameManager (refs: 15)
├── → PlayerService (使用)
├── → CombatSystem (使用)
├── → UIManager (使用)
└── ← SceneLoader (呼び出される)
```

## AI Agentとの連携

### Claude Code
```bash
# CLAUDE.mdまたはプロジェクトコンテキストに追加：
「機能を実装する前に、.repomap/output/ を読んでコードベース構造を理解してください。」
```

### Cursor / Copilot
`.repomap/output/` をプロジェクトのAIコンテキストに追加。

### プロンプト例
> 「L1コードマップを見てモジュール構造を理解して。
> 次にL2でPlayerServiceのシグネチャを確認。
> プレイヤーインベントリを処理する新しいメソッドを実装して。」

## 設定

`.repomap/config.yaml` を編集：

```yaml
project_name: "マイゲーム"

source:
  root_path: "Assets/Scripts"
  exclude_patterns:
    - "**/Editor/**"
    - "**/Tests/**"

# 各レイヤーのトークン予算
tokens:
  l1_skeleton: 1000
  l2_signatures: 2000
  l3_relations: 3000

# 重要なクラスパターンの重みを上げる
importance_boost:
  patterns:
    - prefix: "S"           # SPlayerService → ブースト
      boost: 2.0
    - suffix: "Manager"     # GameManager → ブースト
      boost: 1.5
```

## プリセット

### Unityプリセット
- パス: `Assets/Scripts`
- ブースト: `SXxx` サービスクラス
- カテゴリ: Core、Game、UI、Data、Network、Audio

### 汎用プリセット
- パス: `src`
- ブースト: `Service`、`Repository`、`Controller`
- カテゴリ: Core、Domain、Application、API、Data

## なぜPageRank？

すべてのクラスが同等に重要ではありません。PageRankは参照グラフを分析して**本当に重要な**クラスを特定：

```
高PageRank（重要）：
  - 多くの他クラスから参照される
  - アーキテクチャの中心
  - AIが最初に知るべき

低PageRank（周辺）：
  - ユーティリティクラス、DTO
  - 必要に応じて発見可能
  - トークンを無駄にしない
```

## システム要件

- Python 3.8+
- Git（hooksとコミット情報用）
- Windows 10+ / macOS / Linux

## コントリビュート

コントリビュート歓迎！Pull Requestをお送りください。

## ライセンス

MIT License - [LICENSE](LICENSE) を参照

## 作者

[Yoji](https://github.com/sputnicyoji) 作成

---

**AIコーディングワークフローに役立ったらスターをお願いします！**
