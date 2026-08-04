---
name: conoha-vps-mcp-chatgpt
description: ConoHa VPS MCP ChatGPT 専用エンドポイント（/vps/chatgpt）の操作ガイド。サーバー起動・停止・再起動・削除、ボリューム管理（作成除く）、セキュリティグループ設定など、ChatGPT から ConoHa VPS API を MCP ツールで操作する際に参照する。サーバー作成・ボリューム作成・リサイズは本エンドポイントでは利用不可。「ChatGPT」「vps/chatgpt」「ConoHa」「VPS」「サーバー削除」「サーバー起動」「サーバー停止」「ボリューム」「セキュリティグループ」「conoha_get」「conoha_post」「conoha_delete_by_param」「SSHキーペア」などのキーワードで発動する。
---

# ConoHa VPS MCP 操作ガイド（ChatGPT 専用エンドポイント）

## 前提条件

- MCP クライアントが ConoHa VPS MCP の ChatGPT 専用エンドポイント `/vps/chatgpt` に接続済みで、OAuth 認可が完了していること
- Keystone token / tenant ID は MCP サーバーが OAuth アクセストークンのイントロスペクション結果から自動取得する。ツール利用者が OpenStack の認証情報（ユーザーID・パスワード）や tenant ID を直接指定する必要はない
- API リージョン・接続先はサーバー側の OpenStack base URL 設定で決まる

## このエンドポイントの制限事項

Apps in ChatGPT のストアポリシー（デジタル製品の購入に伴う機能は申請対象外）に対応するため、**料金が発生する操作は本エンドポイントでは恒久的に無効化されている**。

利用不可の操作:

| 操作                   | 標準エンドポイントでの対応ツール                            | 本エンドポイントでの状態 |
| ---------------------- | ----------------------------------------------------------- | ------------------------ |
| サーバー作成           | `conoha_post` path=`/servers`                               | 利用不可                 |
| ボリューム作成         | `conoha_post` path=`/volumes`                               | 利用不可                 |
| リサイズ / 確定 / 取消 | `conoha_post_put_by_param` path=`/action`（resize 系 3 種） | 利用不可                 |
| サーバー作成ウィザード | `create_server` プロンプト                                  | 未登録                   |

**ユーザーからこれらの操作を依頼された場合の対応**:

1. 本エンドポイントでは課金が発生する操作（新規リソースのプロビジョニング・プラン変更）が利用できないことを説明する
2. 代替手段として ConoHa コントロールパネル（<https://manage.conoha.jp/>）、または標準エンドポイント（`/vps/mcp`）に接続した MCP クライアントでの操作を案内する
3. これらの操作をツールで試行しない（スキーマで拒否され、ランタイムガードでもエラーになる）

上記以外の操作はすべて利用可能: 電源操作（起動/停止/再起動）、リモートコンソール、SSHキーペア作成、セキュリティグループ/ルール作成、セキュリティグループ/ポート/ボリューム更新、既存ボリュームのアタッチ（課金なし）、参照系すべて、削除系すべて（削除は課金停止方向のため制限なし）。

## ツール概要

| ツール名                   | HTTPメソッド | 概要                                                |
| -------------------------- | ------------ | --------------------------------------------------- |
| `fetch_url`                | —            | 指定URLからコンテンツを取得                         |
| `encode_base64`            | —            | 文字列をBase64エンコード（1-10000文字）             |
| `conoha_get`               | GET          | リソース一覧取得（10パス）                          |
| `conoha_get_by_param`      | GET          | パラメータ指定で個別リソース取得（6パス）           |
| `conoha_post`              | POST         | リソース作成（3パス）                               |
| `conoha_post_put_by_param` | POST/PUT     | リソース更新・操作（6パス、`/action` は電源系のみ） |
| `conoha_delete_by_param`   | DELETE       | リソース削除（5パス、`confirm: true` 必須）         |

パス・パラメータの詳細は [tool-path-reference.md](references/tool-path-reference.md) を参照。

## 絶対遵守制約

1. **課金操作の試行禁止** — サーバー作成・ボリューム作成・リサイズは依頼されてもツールで試行しない。制限を説明し、ConoHa コントロールパネルへ誘導する
2. **ポート範囲自動設定禁止** — `port_range_min` / `port_range_max` は必ずユーザーに確認して指定する
3. **削除は明示確認必須** — `conoha_delete_by_param` は必ず `confirm: true` を指定する
4. **名前タグ制約** — ボリューム名は英数字・アンダースコア・ハイフンのみ（1-255文字）。SSHキーペア名は英数字・アンダースコア・ハイフンのみ（1文字以上、上限なし）

## ワークフロー判定ツリー

### サーバー操作フロー

| 操作       | ツール                     | path               | requestBody                                                      |
| ---------- | -------------------------- | ------------------ | ---------------------------------------------------------------- |
| 起動       | `conoha_post_put_by_param` | `/action`          | `{"os-start": null}`                                             |
| 停止       | `conoha_post_put_by_param` | `/action`          | `{"os-stop": null}`                                              |
| 強制停止   | `conoha_post_put_by_param` | `/action`          | `{"os-stop": {"force_shutdown": true}}`                          |
| 再起動     | `conoha_post_put_by_param` | `/action`          | `{"reboot": {"type": "SOFT"}}` or `"HARD"`                       |
| コンソール | `conoha_post_put_by_param` | `/remote-consoles` | `{"remote_console": {"protocol": "vnc", "type": "novnc"}}`       |
| 削除       | `conoha_delete_by_param`   | `/servers`         | requestBody なし（flat 入力: param=サーバーID, `confirm: true`） |

※ リサイズ（resize / confirmResize / revertResize）は本エンドポイントでは利用不可。

### セキュリティグループフロー

```text
1. conoha_post path="/v2.0/security-groups" → セキュリティグループ作成
2. conoha_post path="/v2.0/security-group-rules" → ルール追加（ポート範囲はユーザーに確認）
3. conoha_get path="/v2.0/ports" → ポート一覧取得、対象サーバーのポートIDを特定
4. conoha_post_put_by_param path="/v2.0/ports" → ポートにセキュリティグループを適用
```

### ボリューム管理フロー

```text
■ ボリューム作成
  本エンドポイントでは利用不可（課金操作）。ConoHa コントロールパネルへ誘導する

■ ボリュームアタッチ（既存ボリュームの接続。課金なし）
  1. conoha_get path="/volumes/detail" → 既存ボリューム一覧からアタッチ対象の ID を特定
  2. conoha_post_put_by_param path="/os-volume_attachments" param=サーバーID requestBody={"volumeAttachment": {"volumeId": "<ID>"}}

■ ボリューム更新
  conoha_post_put_by_param path="/volumes" param=ボリュームID requestBody={"volume": {"name": "...", "description": "..."}}

■ ボリューム削除
  conoha_delete_by_param path="/volumes" param=ボリュームID confirm=true
```

### 情報取得フロー

| 取得対象                 | ツール                | path                    | param      |
| ------------------------ | --------------------- | ----------------------- | ---------- |
| サーバー一覧             | `conoha_get`          | `/servers/detail`       | —          |
| フレーバー一覧           | `conoha_get`          | `/flavors/detail`       | —          |
| イメージ一覧             | `conoha_get`          | `/v2/images?limit=200`  | —          |
| ボリューム一覧           | `conoha_get`          | `/volumes/detail`       | —          |
| SSHキーペア一覧          | `conoha_get`          | `/os-keypairs`          | —          |
| セキュリティグループ一覧 | `conoha_get`          | `/v2.0/security-groups` | —          |
| ポート一覧               | `conoha_get`          | `/v2.0/ports`           | —          |
| サーバーのIP             | `conoha_get_by_param` | `/ips`                  | サーバーID |
| CPU使用率                | `conoha_get_by_param` | `/rrd/cpu`              | サーバーID |
| ディスク使用率           | `conoha_get_by_param` | `/rrd/disk`             | サーバーID |

## ユーザー発話パターンとツール対応

| 発話パターン                       | 使用ツール                 | パス                                                   |
| ---------------------------------- | -------------------------- | ------------------------------------------------------ |
| 「サーバーを作成して」             | —（対応不可）              | 制限を説明し ConoHa コントロールパネルへ誘導           |
| 「サーバーをリサイズして」         | —（対応不可）              | 制限を説明し ConoHa コントロールパネルへ誘導           |
| 「ボリュームを作成して」           | —（対応不可）              | 制限を説明し ConoHa コントロールパネルへ誘導           |
| 「サーバーを停止/起動/再起動して」 | `conoha_post_put_by_param` | `/action`                                              |
| 「サーバーを削除して」             | `conoha_delete_by_param`   | `/servers`（`confirm: true` 必須）                     |
| 「セキュリティグループを作成して」 | `conoha_post`              | `/v2.0/security-groups` + `/v2.0/security-group-rules` |
| 「サーバーの状態を確認して」       | `conoha_get`               | `/servers/detail`                                      |
| 「コンソールに接続して」           | `conoha_post_put_by_param` | `/remote-consoles`                                     |

## エラー対応ガイド

| エラー                                                                | 原因                                                | 対処                                                                |
| --------------------------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------------- |
| 「この操作は当エンドポイントでは無効です (料金が発生する操作のため)」 | 課金操作（サーバー/ボリューム作成・リサイズ）の試行 | 制限を説明し ConoHa コントロールパネルへ誘導                        |
| 401 Unauthorized                                                      | OAuth 認可切れ、scope 不足、Bearer token 不正       | MCP クライアント側で再認可し、`vps:read` / `vps:write` scope を確認 |
| 409 Conflict                                                          | リソース競合（削除中のボリューム等）                | 状態を確認して再試行                                                |
| 400 Bad Request (port_range)                                          | ポート範囲不正                                      | 0-65535の整数か確認                                                 |
| 404 Not Found                                                         | リソースが存在しない                                | ID/名前を再確認                                                     |

## リファレンス

- [ツール別パス・パラメータ一覧](references/tool-path-reference.md)
- [リクエストボディスキーマ全集](references/request-body-schemas.md)
- [主要操作のワークフローレシピ](references/workflow-recipes.md)
