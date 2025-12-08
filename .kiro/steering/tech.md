# 技術スタック

## アーキテクチャ
Kiro Power - MCPツールとフレームワーク専門知識を統合したKiroの拡張機能

## Powerの構成要素

このPowerは以下の要素で構成される：

1. **MCPツール**: Home Assistant APIへの直接アクセスを提供する80以上のツール
2. **フレームワーク専門知識**: POWER.mdとSteeringファイルによるHome Assistantのパターン、ベストプラクティス、ワークフローの知識
3. **動的コンテキストロード**: キーワードに基づいて関連するツールとコンテキストのみを読み込む

## 主要技術

### MCPサーバー
- **パッケージ**: `ha-mcp@latest`
- **実行環境**: uvx (uv package manager)
- **認証**: Home Assistant長期アクセストークン
- **役割**: Home Assistant APIへのアクセスとデバイス制御

### 必須依存関係
- **uv**: Pythonパッケージマネージャー（uvxコマンドの実行に必要）
  - macOS/Linux: `brew install uv` または `curl -LsSf https://astral.sh/uv/install.sh | sh`
  - Windows: `winget install astral-sh.uv -e`

### 環境変数
- `HOMEASSISTANT_URL`: Home AssistantのURL（例: `http://homeassistant.local:8123`）
- `HOMEASSISTANT_TOKEN`: 長期アクセストークン
- `FASTMCP_LOG_LEVEL`: ログレベル（デフォルト: ERROR）

## プロジェクト構成ファイル

### mcp.json
MCPサーバーの設定ファイル。uvxコマンドで`ha-mcp@latest`パッケージを実行し、環境変数を設定する。

### POWER.md
Kiro Powerのエントリーポイント。以下を含む：
- フロントマター: name、displayName、description、keywords、author
- 機能説明: MCPツールの概要と使用方法
- ベストプラクティス: Home Assistant開発のパターンと推奨事項
- ワークフロー: 一般的な使用パターン
- トラブルシューティング: 一般的な問題と解決方法

### steering/
Power使用時に追加のコンテキストと指示を提供するSteeringファイル群：
- `homeassistant-dev-guide.md`: Home Assistant開発ガイド（YAML構文、自動化パターン、ベストプラクティス）
- `homeassistant-mcp-tools.md`: MCPツールリファレンス（各ツールの詳細な使用方法）

### README.md
ユーザー向けドキュメント。インストール方法、設定、使用例、トラブルシューティングを記載。

## Powerの動作原理

### キーワードベースの活性化
POWER.mdのフロントマターに定義されたキーワード（例: "homeassistant", "hass", "ha-mcp"）に基づいて、Powerが自動的に活性化される。

### 動的ツールロード
従来のMCP実装では全ツールを事前にロードするが、Powerは必要なツールのみを動的にロード：
- ユーザーが「automation」に言及 → 自動化関連のツールとコンテキストをロード
- ユーザーが「dashboard」に言及 → ダッシュボード関連のツールとコンテキストをロード
- 不要なツールは非活性化され、コンテキストウィンドウを節約

### コンテキスト管理
- POWER.md: 常に読み込まれる基本コンテキスト
- Steeringファイル: 必要に応じて動的に読み込まれる専門コンテキスト

## 開発規約

### ドキュメント
- README.mdとPOWER.mdは英語で記述する（MUST）
- Steeringファイルは英語で記述する（MUST）
- ユーザー向けドキュメントは明確で実用的な例を含む（SHOULD）

### 環境変数の扱い
- トークンは機密情報として扱う（MUST）
- コードやドキュメントにトークンをハードコードしてはならない（MUST NOT）
- 環境変数を使用する（MUST）

### Powerの設計原則
- MCPツールとフレームワーク知識を統合する（MUST）
- 単なるAPI アクセスではなく、専門知識を提供する（MUST）
- YAML設定作成、デバッグ、ベストプラクティスの適用を支援する（SHOULD）

## 公式Powerの構造パターン

公式Kiro Powersリポジトリ（https://github.com/kirodotdev/powers）の構造に基づく：

### ディレクトリ構成
```
power-name/
├── POWER.md      # メインドキュメント（フロントマター + 内容）
├── mcp.json      # MCPサーバー設定
└── steering/     # Steeringファイル（オプション）
    └── *.md
```

### POWER.mdフロントマター
```yaml
---
name: "power-name"           # 識別子（小文字、ハイフン）
displayName: "Display Name"  # 表示名
description: "Description"   # 説明
keywords: ["keyword1", "keyword2"]  # 活性化キーワード
author: "Author Name"        # 作者
---
```

### キーワード設計のガイドライン
キーワードは一般的すぎると予期せぬところでPowerがトリガーされる可能性がある。

**避けるべきキーワード（一般的すぎる）:**
- `automation` - CI/CD、テスト自動化などでもトリガーされる
- `yaml` - あらゆるYAML設定でトリガーされる
- `dashboard` - Grafana、Kibanaなど他のダッシュボードでも
- `script` - シェルスクリプト、Pythonスクリプトなどでも
- `scene` - 3Dモデリング、ゲーム開発などでも
- `smart home` - 一般的なIoT議論でも
- `iot` - 非常に広範な用語

**推奨されるキーワード（固有性が高い）:**
- 製品/サービス固有の名前（例: `homeassistant`, `home assistant`）
- 公式の略称（例: `hass`）
- パッケージ名（例: `ha-mcp`）
- 製品固有の機能名（例: `lovelace`）

### mcp.json構造
```json
{
  "mcpServers": {
    "Server Name": {
      "command": "uvx",
      "args": ["package@latest"],
      "env": { ... }
    }
  }
}
```

または（リモートMCPサーバーの場合）:
```json
{
  "mcpServers": {
    "Server Name": {
      "url": "https://mcp.example.com"
    }
  }
}
```

# 外部リソース

### Kiro Power

- [Create poowers](https://kiro.dev/docs/powers/create/)
- [Kiro Powers Repository](https://github.com/kirodotdev/powers)
- [Introducing Kiro powers - Blog Article](https://kiro.dev/blog/introducing-powers/)

### Home Assistant MCP Server

- [Home Assistant MCP Server](https://github.com/homeassistant-ai/ha-mcp)
- [FAQ & Troubleshooting](https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/FAQ.md)
- [macOS uv Installation](https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/macOS-uv-guide.md)
- [Windows uv Installation](https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/Windows-uv-guide.md)

### Home Assistant Developer Guide

- [Standards](https://developers.home-assistant.io/docs/documenting/standards)
- [General style guide](https://developers.home-assistant.io/docs/documenting/general-style-guide)
- [YAML Style Guide](https://developers.home-assistant.io/docs/documenting/yaml-style-guide/)
