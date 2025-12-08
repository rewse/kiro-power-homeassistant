# プロジェクト構造

## ディレクトリ構成

```
.
├── power-homeassistant/   # Powerファイル格納ディレクトリ
│   ├── POWER.md          # Kiro Powerメタデータと詳細ドキュメント
│   └── mcp.json          # MCPサーバー設定
├── LICENSE.md             # ライセンスファイル
└── README.md              # ユーザー向けドキュメント
```

## ファイルの役割

### power-homeassistant/POWER.md
- Kiro Powerのメインドキュメント
- フロントマターにメタデータを含む（name、displayName、description、keywords、author、mcpServers）
- 機能説明、使用方法、ベストプラクティス、トラブルシューティングを記載
- 英語で記述する（MUST）

### power-homeassistant/mcp.json
- MCPサーバーの設定ファイル
- サーバー名、コマンド、引数、環境変数を定義
- JSON形式（コメント可）

## 命名規則

### ファイル名
- 小文字とハイフンを使用（例: `product.md`、`tech.md`）
- 拡張子は適切に使用（`.md`、`.json`）

### MCPサーバー名
- mcp.jsonでは表示名を使用（例: "Home Assistant"）
- POWER.mdのmcpServersフィールドでは識別子を使用（例: "homeassistant"）

## ベストプラクティス

### ドキュメントの更新
- power/POWER.mdとREADME.mdの両方を更新する（SHOULD）

### ファイルの配置
- Power関連ファイル（POWER.md、mcp.json）はpower-homeassistant/ディレクトリに配置する（MUST）。[Kiro Power 作成ガイド](https://kiro.dev/docs/powers/create/] ではプロジェクトルートに配置しているが、そうするとREADME.mdや.gitignoreを配置できなくなる
- ユーザー向けドキュメント（README.md、LICENSE.md）はルートに配置する（MUST）
