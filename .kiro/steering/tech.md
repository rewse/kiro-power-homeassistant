# 技術スタック

## アーキテクチャ
Kiro Power - MCPサーバーを使用したKiroの拡張機能

## 主要技術

### MCPサーバー
- **パッケージ**: `@homeassistant-ai/ha-mcp@latest`
- **実行環境**: uvx (uv package manager)
- **認証**: Home Assistant長期アクセストークン

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
MCPサーバーの設定ファイル。uvxコマンドでha-mcpパッケージを実行し、環境変数を設定する。

### POWER.md
Kiro Powerのメタデータと詳細ドキュメント。フロントマターにname、displayName、description、keywords、author、mcpServersを含む。

### README.md
ユーザー向けドキュメント。インストール方法、設定、使用例、トラブルシューティングを記載。

## 開発規約

### ドキュメント
- README.mdとPOWER.mdは英語で記述する（MUST）

### 環境変数の扱い
- トークンは機密情報として扱う（MUST）
- コードやドキュメントにトークンをハードコードしてはならない（MUST NOT）
- 環境変数を使用する（MUST）

## 外部リソース
- **Kiro Power 作成ガイド**: https://kiro.dev/docs/powers/create/
- **Kiro Powers リポジトリ**: https://github.com/kirodotdev/powers
- **MCPサーバーソース**: https://github.com/homeassistant-ai/ha-mcp
- **MCPサーバーFAQ**: https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/FAQ.md
- **macOS uv インストールガイド**: https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/macOS-uv-guide.md
- **Windows uv インストールガイド**: https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/Windows-uv-guide.md
