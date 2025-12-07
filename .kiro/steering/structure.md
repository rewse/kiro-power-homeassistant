# プロジェクト構造

## ディレクトリ構成

```
.
├── .kiro/                  # Kiro設定ディレクトリ
│   └── steering/          # ステアリングルール（このファイルを含む）
├── LICENSE.md             # ライセンスファイル
├── POWER.md               # Kiro Powerメタデータと詳細ドキュメント
├── README.md              # ユーザー向けドキュメント
└── mcp.json               # MCPサーバー設定
```

## ファイルの役割

### POWER.md
- Kiro Powerのメインドキュメント
- フロントマターにメタデータを含む（name、displayName、description、keywords、author、mcpServers）
- 機能説明、使用方法、ベストプラクティス、トラブルシューティングを記載
- 英語で記述する（MUST）

### README.md
- ユーザー向けの簡潔なドキュメント
- インストール手順、前提条件、設定方法、使用例を記載
- 英語で記述する（MUST）

### mcp.json
- MCPサーバーの設定ファイル
- サーバー名、コマンド、引数、環境変数を定義
- JSON形式（コメント可）

### LICENSE.md
- プロジェクトのライセンス情報

### .kiro/steering/
- Kiroのステアリングルールを格納
- Markdownファイルで記述
- 常に含まれる（デフォルト動作）

## 命名規則

### ファイル名
- 小文字とハイフンを使用（例: `product.md`、`tech.md`）
- 拡張子は適切に使用（`.md`、`.json`）

### MCPサーバー名
- mcp.jsonでは表示名を使用（例: "Home Assistant"）
- POWER.mdのmcpServersフィールドでは識別子を使用（例: "homeassistant"）


### ドキュメントの更新
- POWER.mdとREADME.mdの両方を更新する（SHOULD）
- 一貫性を保つ（MUST）
- 英語で記述する（MUST）
