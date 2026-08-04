# リクエストボディスキーマ全集（ChatGPT 専用エンドポイント）

`CreateServerRequest`（`/servers`）と `CreateVolumeRequest`（`/volumes`）は課金が発生する操作のため本エンドポイントでは利用できず、本書にも含めない。

## conoha_post 用スキーマ

### CreateSSHKeyPairRequest（path: `/os-keypairs`）

```json
{
	"keypair": {
		"name": "<string>",
		"public_key": "<string>"
	}
}
```

| フィールド   | 型     | 必須 | 説明                                                                    |
| ------------ | ------ | ---- | ----------------------------------------------------------------------- |
| `name`       | string | 必須 | SSHキーペアの名前（英数字・`_`・`-`のみ、正規表現: `^[A-Za-z0-9_-]+$`） |
| `public_key` | string | 必須 | SSHキーペアの公開鍵                                                     |

**注意**: `public_key` は必須。省略すると新しいキーペアが生成され、秘密鍵がレスポンスに含まれてしまうため、省略不可とされている。

---

### CreateSecurityGroupRequest（path: `/v2.0/security-groups`）

```json
{
	"security_group": {
		"name": "<string>",
		"description": "<string>"
	}
}
```

| フィールド    | 型     | 必須 | 説明                       |
| ------------- | ------ | ---- | -------------------------- |
| `name`        | string | 必須 | セキュリティグループの名前 |
| `description` | string | 任意 | セキュリティグループの説明 |

---

### CreateSecurityGroupRuleRequest（path: `/v2.0/security-group-rules`）

```json
{
	"security_group_rule": {
		"security_group_id": "<string>",
		"direction": "ingress",
		"ethertype": "IPv4",
		"port_range_min": 80,
		"port_range_max": 80,
		"protocol": "tcp",
		"remote_ip_prefix": "0.0.0.0/0",
		"remote_group_id": "<string>"
	}
}
```

| フィールド          | 型                                       | 必須 | 説明                                                       |
| ------------------- | ---------------------------------------- | ---- | ---------------------------------------------------------- |
| `security_group_id` | string                                   | 必須 | セキュリティグループのID                                   |
| `direction`         | `"ingress"` \| `"egress"`                | 必須 | ルールの方向                                               |
| `ethertype`         | `"IPv4"` \| `"IPv6"`                     | 必須 | イーサタイプ。サーバー側デフォルトはないため必ず指定する   |
| `port_range_min`    | number (0-65535)                         | 任意 | 最小ポート番号。**ユーザーに確認して指定。自動設定禁止。** |
| `port_range_max`    | number (0-65535)                         | 任意 | 最大ポート番号。**ユーザーに確認して指定。自動設定禁止。** |
| `protocol`          | `"tcp"` \| `"udp"` \| `"icmp"` \| `null` | 任意 | プロトコル                                                 |
| `remote_ip_prefix`  | string                                   | 任意 | リモートIPのCIDR（例: `"0.0.0.0/0"`）                      |
| `remote_group_id`   | string                                   | 任意 | リモートセキュリティグループのID                           |

---

## conoha_post_put_by_param 用スキーマ

### OperateServerPowerRequest（path: `/action`, param: サーバーID）

電源系の4つのバリアントから1つを選択。標準エンドポイントで利用できるリサイズ系バリアント（`resize` / `confirmResize` / `revertResize`）は課金が発生するため本エンドポイントでは受け付けない:

| バリアント | requestBody                              | 説明                                |
| ---------- | ---------------------------------------- | ----------------------------------- |
| 起動       | `{"os-start": null}`                     | サーバーを起動する                  |
| 停止       | `{"os-stop": null}`                      | サーバーを停止する                  |
| 強制停止   | `{"os-stop": {"force_shutdown": true}}`  | サーバーを強制シャットダウンする    |
| 再起動     | `{"reboot": {"type": "SOFT" \| "HARD"}}` | サーバーを再起動する（SOFT / HARD） |

---

### RemoteConsoleRequest（path: `/remote-consoles`, param: サーバーID）

```json
{
	"remote_console": {
		"protocol": "vnc",
		"type": "novnc"
	}
}
```

| フィールド | 型                               | 必須 | 説明                           |
| ---------- | -------------------------------- | ---- | ------------------------------ |
| `protocol` | `"vnc"` \| `"serial"` \| `"web"` | 必須 | リモートコンソールのプロトコル |
| `type`     | `"novnc"` \| `"serial"`          | 必須 | リモートコンソールのタイプ     |

**注意**: protocol と type の組み合わせは一致させる必要がある（例: vnc + novnc, serial + serial）。

---

### AttachVolumeRequest（path: `/os-volume_attachments`, param: サーバーID）

```json
{
	"volumeAttachment": {
		"volumeId": "<string>"
	}
}
```

| フィールド | 型     | 必須 | 説明                       |
| ---------- | ------ | ---- | -------------------------- |
| `volumeId` | string | 必須 | アタッチするボリュームのID |

**注意**: 既存ボリュームの接続は課金が発生しないため本エンドポイントでも利用できる（新規ボリュームの作成は不可）。

---

### UpdateSecurityGroupRequest（path: `/v2.0/security-groups`, param: セキュリティグループID）

```json
{
	"security_group": {
		"name": "<string>",
		"description": "<string>"
	}
}
```

| フィールド    | 型     | 必須 | 説明                       |
| ------------- | ------ | ---- | -------------------------- |
| `name`        | string | 任意 | セキュリティグループの名前 |
| `description` | string | 任意 | セキュリティグループの説明 |

---

### UpdatePortRequest（path: `/v2.0/ports`, param: ポートID）

```json
{
	"port": {
		"security_groups": ["<セキュリティグループID>"]
	}
}
```

| フィールド        | 型       | 必須 | 説明                                             |
| ----------------- | -------- | ---- | ------------------------------------------------ |
| `security_groups` | string[] | 任意 | ポートに関連付けるセキュリティグループIDのリスト |

---

### UpdateVolumeRequest（path: `/volumes`, param: ボリュームID）

```json
{
	"volume": {
		"name": "<string>",
		"description": "<string>"
	}
}
```

| フィールド    | 型     | 必須 | 説明                                            |
| ------------- | ------ | ---- | ----------------------------------------------- |
| `name`        | string | 任意 | ボリューム名（1-255文字、英数字・`_`・`-`のみ） |
| `description` | string | 任意 | ボリュームの説明                                |
