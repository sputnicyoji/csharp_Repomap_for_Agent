# csharp-repomap

[![PyPI version](https://badge.fury.io/py/csharp-repomap.svg)](https://badge.fury.io/py/csharp-repomap)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**[English](README.md)** | **[简体中文](README.zh-CN.md)** | **日本語**

> C#プロジェクト向けの階層化コードマップを生成。Claude等のAIアシスタントに最適化。

## これは何？

**csharp-repomap** は、AIアシスタントがC#コードベースを理解するための構造化されたコードマップを作成します。3つの詳細レベルを生成します：

| レベル | トークン数 | 内容 |
|--------|-----------|------|
| **L1 スケルトン** | ~1k | モジュール概要、カテゴリ、コアエントリクラス |
| **L2 シグネチャ** | ~2k | 重要クラスとメソッドシグネチャ |
| **L3 リレーション** | ~3k | 参照グラフ（誰が誰を呼ぶか） |

このツールは **tree-sitter** を使用して正確なC#解析を行い、**PageRank** アルゴリズムでコードベース内の最も重要なクラスを特定します。

## なぜ必要？

AIアシスタントのコンテキストウィンドウには制限があります。大規模なコードベース（1000+ファイル）を扱う場合、全体像を把握できません。**csharp-repomap** は以下の方法でこの問題を解決します：

1. **重要なコードを優先** - PageRankがコアクラスを特定
2. **階層化された詳細** - スケルトンから始めて、必要に応じて掘り下げ
3. **トークン意識** - コンテキスト制限に適合
4. **自動更新** - Git hooksでマップを最新に保持

## 機能

- **Tree-sitter解析** - 正確なC#構文解析
- **PageRankランキング** - 参照カウントで重要クラスを特定
- **トークン制限出力** - AIコンテキストウィンドウに適合
- **Git hooks** - pull/merge/checkout時に自動更新
- **クロスプラットフォーム通知** - Windows Toast、macOS、Linux
- **Unityプリセット** - Unityプロジェクト用に事前設定
- **汎用プリセット** - 任意のC#プロジェクトに対応

## インストール

```bash
pip install csharp-repomap
```

トークンカウント機能（オプション）：
```bash
pip install csharp-repomap[tiktoken]
```

## クイックスタート

```bash
# プロジェクトで初期化（プリセットを選択）
cd your-csharp-project
repomap init --preset unity    # Unityプロジェクト
repomap init --preset generic  # その他のC#プロジェクト

# リポマップを生成
repomap generate --verbose

# ステータスを確認
repomap status

# Git hooksをインストールして自動更新
repomap hooks --install
```

## 設定

`repomap init` 実行後、`.repomap/config.yaml` を編集：

```yaml
project_name: "マイプロジェクト"

source:
  root_path: "Assets/Scripts"  # C#ソースパス
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

## 出力ファイル

`.repomap/output/` ディレクトリに生成：

| ファイル | 説明 |
|----------|------|
| `repomap-L1-skeleton.md` | モジュール概要、カテゴリ、コアクラス |
| `repomap-L2-signatures.md` | 重要クラスとメソッドシグネチャ |
| `repomap-L3-relations.md` | 参照グラフ |
| `repomap-meta.json` | Git情報、統計、タイムスタンプ |

## Git Hooks

コード変更時にマップを自動更新：

```bash
# hooksをインストール
repomap hooks --install

# hooksをアンインストール
repomap hooks --uninstall
```

トリガータイミング：
- `git pull`
- `git merge`
- `git checkout`（ブランチ切り替え）

## Claude Codeとの連携

1. `.repomap/output/` をプロジェクトコンテキストに追加
2. ClaudeがL1/L2/L3ファイルを見てコードベース構造を理解
3. 新しいコードをpullするとマップが自動更新

プロンプト例：
> "リポマップを見てモジュール構造を理解してから、実装して..."

## プリセット

### Unityプリセット
- `Assets/Scripts` パス用に設定
- `SXxx` サービスクラスの重みを上げる
- カテゴリ：Core、Game、UI、Data、Network、Audio

### 汎用プリセット
- `src` ディレクトリ用に設定
- `Service`、`Repository`、`Controller` パターンの重みを上げる
- カテゴリ：Core、Domain、Application、API、Data

## システム要件

- Python 3.8+
- Git（hooksとコミット情報用）
- Windows 10+ / macOS / Linux（通知用）

## コントリビュート

コントリビュート歓迎！お気軽にPull Requestを送ってください。

## ライセンス

MIT License - [LICENSE](LICENSE) を参照

## 作者

[Yoji](https://github.com/sputnicyoji) 作成

---

**役に立ったらスターをお願いします！**
