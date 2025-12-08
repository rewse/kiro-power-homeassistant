# プロジェクト構造

## ディレクトリ構成

```
.
├── power-homeassistant/   # Powerファイル格納ディレクトリ
│   ├── POWER.md          # Kiro Powerメタデータと詳細ドキュメント
│   ├── mcp.json          # MCPサーバー設定
│   └── steering/         # Kiro Steering ファイル格納ディレクトリ
│       ├── homeassistant-dev-guide.md    # Home Assistant開発ガイド
│       ├── homeassistant-mcp-tools.md    # MCPツールリファレンス
│       └── homeassistant-scripts-guide.md # スクリプトガイド
├── LICENSE.md             # ライセンスファイル
└── README.md              # ユーザー向けドキュメント
```

## ファイルの役割

### power-homeassistant/POWER.md
- Kiro Powerのメインドキュメント
- フロントマターにメタデータを含む（name、displayName、description、keywords、author）
- 機能説明、使用方法、ベストプラクティス、トラブルシューティングを記載
- 英語で記述する（MUST）

### power-homeassistant/mcp.json
- MCPサーバーの設定ファイル
- サーバー名、コマンド、引数、環境変数を定義
- JSON形式

### power-homeassistant/steering/
- Kiro Steeringファイルを格納するディレクトリ
- Powerの使用時に追加のコンテキストと指示を提供
- 各ファイルは特定の目的を持つ（開発ガイド、ツールリファレンスなど）
- Markdown形式で記述する（MUST）

## 公式Powerの構造パターン

公式Kiro Powersリポジトリ（https://github.com/kirodotdev/powers）に基づく標準構造：

```
power-name/
├── POWER.md      # メインドキュメント
├── mcp.json      # MCPサーバー設定
└── steering/     # Steeringファイル（オプション）
    └── *.md
```

### 公式Powerの例
- **stripe**: POWER.md + mcp.json + steering/
- **neon**: POWER.md + mcp.json + steering/steering.md
- **cloud-architect**: POWER.md + mcp.json + steering/（複数ファイル）
- **figma**: POWER.md + mcp.json（steeringなし）

## 命名規則

### ファイル名
- 小文字とハイフンを使用（例: `homeassistant-dev-guide.md`）
- 拡張子は適切に使用（`.md`、`.json`）

### MCPサーバー名
- mcp.jsonでは表示名を使用（例: "Home Assistant"）
- POWER.mdのnameフィールドでは識別子を使用（例: "homeassistant"）

## ベストプラクティス

### ドキュメントの更新
- power-homeassistant/POWER.mdとREADME.mdの両方を更新する（SHOULD）

### ファイルの配置
- Power関連ファイル（POWER.md、mcp.json）はpower-homeassistant/ディレクトリに配置する（MUST）
- Steeringファイルはpower-homeassistant/steering/ディレクトリに配置する（MUST）
- ユーザー向けドキュメント（README.md、LICENSE.md）はルートに配置する（MUST）

### Steeringファイルの管理
- 各Steeringファイルは単一の責任を持つ（SHOULD）
- ファイル名は内容を明確に表す（MUST）
- 開発ガイドとツールリファレンスは分離する（SHOULD）

### POWER.mdの構成
公式Powerに基づく推奨構成：
1. フロントマター（name、displayName、description、keywords、author）
2. Overview（概要と主要機能）
3. Available Steering Files（利用可能なSteeringファイル）
4. Available MCP Servers（MCPサーバー情報）
5. Best Practices（ベストプラクティス）
6. Common Workflows（一般的なワークフロー）
7. Configuration（設定方法）
8. Troubleshooting（トラブルシューティング）
9. Tips（ヒント）
10. Resources（リソース）
