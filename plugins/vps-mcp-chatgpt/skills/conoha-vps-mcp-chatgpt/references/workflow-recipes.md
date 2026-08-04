# 主要操作のワークフローレシピ（ChatGPT 専用エンドポイント）

サーバー作成・ボリューム作成・リサイズのレシピは、課金が発生する操作のため本エンドポイントでは提供しない。依頼された場合は ConoHa コントロールパネル（<https://manage.conoha.jp/>）へ誘導する。

## 1. サーバーの電源操作（起動/停止/再起動）

### Step 1: サーバー一覧確認

ツール: `conoha_get`

```json
{ "path": "/servers/detail" }
```

→ 対象サーバーの `id` と現在の `status` を確認（起動は `SHUTOFF`、停止/再起動は `ACTIVE` が前提）

### Step 2: 電源操作の実行

ツール: `conoha_post_put_by_param`

起動:

```json
{
	"input": {
		"path": "/action",
		"param": "<サーバーID>",
		"requestBody": { "os-start": null }
	}
}
```

停止:

```json
{
	"input": {
		"path": "/action",
		"param": "<サーバーID>",
		"requestBody": { "os-stop": null }
	}
}
```

再起動:

```json
{
	"input": {
		"path": "/action",
		"param": "<サーバーID>",
		"requestBody": { "reboot": { "type": "SOFT" } }
	}
}
```

→ 通常の停止が効かない場合のみ、ユーザーに確認のうえ `{"os-stop": {"force_shutdown": true}}`（強制停止）や `"type": "HARD"`（ハード再起動）を使用する

### Step 3: 状態確認

ツール: `conoha_get`

```json
{ "path": "/servers/detail" }
```

→ status が期待する状態（起動なら `ACTIVE`、停止なら `SHUTOFF`）になったことを確認

---

## 2. セキュリティグループ作成とサーバーへの適用

### Step 1: セキュリティグループ作成

ツール: `conoha_post`

```json
{
	"input": {
		"path": "/v2.0/security-groups",
		"requestBody": {
			"security_group": {
				"name": "web-server-sg",
				"description": "HTTP/HTTPSを許可するセキュリティグループ"
			}
		}
	}
}
```

→ 作成されたセキュリティグループの `id` を控える

### Step 2: ルール追加（HTTP）

ツール: `conoha_post`

```json
{
	"input": {
		"path": "/v2.0/security-group-rules",
		"requestBody": {
			"security_group_rule": {
				"security_group_id": "<セキュリティグループID>",
				"direction": "ingress",
				"ethertype": "IPv4",
				"protocol": "tcp",
				"port_range_min": 80,
				"port_range_max": 80,
				"remote_ip_prefix": "0.0.0.0/0"
			}
		}
	}
}
```

→ **port_range_min / port_range_max はユーザーに確認して設定。自動設定禁止。**

### Step 3: ルール追加（HTTPS）

同様に port_range_min: 443, port_range_max: 443 でルールを追加。

### Step 4: ポート一覧取得

ツール: `conoha_get`

```json
{ "path": "/v2.0/ports" }
```

→ 対象サーバーの `device_id` が一致するポートの `id` を特定

### Step 5: ポートにセキュリティグループを適用

ツール: `conoha_post_put_by_param`

```json
{
	"input": {
		"path": "/v2.0/ports",
		"param": "<ポートID>",
		"requestBody": {
			"port": {
				"security_groups": ["<セキュリティグループID>"]
			}
		}
	}
}
```

---

## 3. 既存ボリュームのアタッチ

新規ボリュームの作成は本エンドポイントでは行えない。アタッチできるのは既存の未接続ボリュームのみ。

### Step 1: ボリューム一覧確認

ツール: `conoha_get`

```json
{ "path": "/volumes/detail" }
```

→ アタッチ対象ボリュームの `id` と `status`（`available` であること）を確認

### Step 2: ボリュームアタッチ

ツール: `conoha_post_put_by_param`

```json
{
	"input": {
		"path": "/os-volume_attachments",
		"param": "<サーバーID>",
		"requestBody": {
			"volumeAttachment": {
				"volumeId": "<ボリュームID>"
			}
		}
	}
}
```

### Step 3: 状態確認

ツール: `conoha_get`

```json
{ "path": "/volumes/detail" }
```

→ ボリュームの status が `in-use` になったことを確認

---

## 4. サーバー削除手順

### Step 1: サーバー一覧確認

ツール: `conoha_get`

```json
{ "path": "/servers/detail" }
```

→ 削除対象サーバーの `id` を確認

### Step 2: サーバー停止（起動中の場合）

ツール: `conoha_post_put_by_param`

```json
{
	"input": {
		"path": "/action",
		"param": "<サーバーID>",
		"requestBody": {
			"os-stop": null
		}
	}
}
```

### Step 3: サーバー削除

ツール: `conoha_delete_by_param`

```json
{
	"path": "/servers",
	"param": "<サーバーID>",
	"confirm": true
}
```

**注意**: サーバー削除は取り消せない。削除前にユーザーに確認すること。
