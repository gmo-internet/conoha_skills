<div align="center">

![ConoHa VPS Logo](assets/conoha_logo.svg)

# ConoHa Skills

</div>

GMO Internet が提供する ConoHa の skills を管理している公式レポジトリです。
ConoHa の各製品を Claude Code から MCP 経由で安全に操作するためのプラグインなどを配布しております。

## マーケットプレイスの追加

最初に一度だけマーケットプレイスを登録します。

```
/plugin marketplace add gmo-internet/conoha_skills
```

## 提供プラグイン一覧

| 製品       | プラグイン | インストール                     | MCP エンドポイント              |
| ---------- | ---------- | -------------------------------- | ------------------------------- |
| ConoHa VPS | `vps-mcp`  | `/plugin install vps-mcp@conoha` | `https://api.conoha.jp/vps/mcp` |

---

## ConoHa VPS プラグイン（`vps-mcp`）

ConoHa VPS を Claude Code から MCP 経由で安全に操作するためのプラグインです。
サーバー / ボリューム / セキュリティグループ操作のワークフロー・制約・リクエストスキーマを
ガイドする Skill と、ConoHa VPS MCP サーバーへの接続を同梱しています。

### 含まれるもの

- **Skill `conoha-vps-mcp`**
  サーバーの作成・削除・起動・停止・リサイズ、ボリューム管理、セキュリティグループ設定などを、
  正しい MCP ツール・手順・制約に沿ってガイドします。以下の参照ドキュメントを同梱しています。
  - `tool-path-reference.md` — 各 MCP ツールで許可される API パス一覧
  - `request-body-schemas.md` — 作成・更新系操作のリクエストボディスキーマ
  - `workflow-recipes.md` — サーバー作成・削除・リサイズ等の手順レシピ
- **MCP サーバー `conoha-vps-mcp`**
  ConoHa VPS API を MCP ツールとして公開する HTTP リモートサーバー
  (`https://api.conoha.jp/vps/mcp`)。プラグイン有効化時に自動で登録されます。

### 前提条件

- プラグイン / マーケットプレイスに対応した Claude Code
- ConoHa アカウントおよび ConoHa VPS の利用契約
- 初回利用時に MCP サーバーの有効化を承認し、ConoHa の認証を完了していること

### インストール

```
/plugin marketplace add gmo-internet/conoha_skills
/plugin install vps-mcp@conoha
```

- インストール後、MCP サーバー `conoha-vps-mcp` の有効化を承認してください。
- 初回のツール実行時に ConoHa の認証フローが実行されます。

### 使い方（発話例）

Skill が自動で発動し、適切なツールと手順・制約に沿って操作をガイドします。

- 「ConoHa でサーバーを作成して」
- 「VPS を一覧表示して」
- 「スタートアップスクリプトでサーバーを立てて」
- 「セキュリティグループを作って対象サーバーに適用して」
- 「サーバーをリサイズして」
- 「サーバーを削除して」

---

## ライセンス

[Apache License 2.0](./LICENSE) — Copyright 2026 GMO Internet

## リンク

- ConoHa: https://www.conoha.jp/
- GMO Internet: https://github.com/gmo-internet
