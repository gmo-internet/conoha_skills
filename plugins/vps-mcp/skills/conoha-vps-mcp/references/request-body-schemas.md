# リクエストボディスキーマ全集

## conoha_post 用スキーマ

### CreateServerRequest（path: `/servers`）

```json
{
	"server": {
		"flavorRef": "<string>",
		"adminPass": "<string>",
		"block_device_mapping_v2": [{ "uuid": "<string>" }],
		"metadata": {
			"instance_name_tag": "<string>" // フィールドは任意。metadata オブジェクト自体は必須
		},
		"security_groups": [
			// 任意
			{ "name": "<string>" }
		],
		"key_name": "<string>", // 任意
		"user_data": "<string>" // 任意
	}
}
```

| フィールド                       | 型     | 必須 | 説明                                                                                                                                                                                        |
| -------------------------------- | ------ | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `flavorRef`                      | string | 必須 | フレーバーID                                                                                                                                                                                |
| `adminPass`                      | string | 必須 | サーバーの管理者/rootパスワード。**ユーザーが指定した値のみを使用。自動設定禁止。**                                                                                                         |
| `block_device_mapping_v2`        | array  | 必須 | ボリュームマッピング。サーバー作成時は起動元ボリュームを指定する                                                                                                                            |
| `block_device_mapping_v2[].uuid` | string | 必須 | 起動元となるボリュームのUUID                                                                                                                                                                |
| `metadata`                       | object | 必須 | サーバーメタデータ。表示名を付けない場合も `{}` を指定する                                                                                                                                  |
| `metadata.instance_name_tag`     | string | 任意 | 表示名（1-255文字、英数字・`_`・`-`のみ、正規表現: `^[A-Za-z0-9_-]{1,255}$`）                                                                                                               |
| `security_groups`                | array  | 任意 | セキュリティグループのリスト                                                                                                                                                                |
| `security_groups[].name`         | string | —    | セキュリティグループ名                                                                                                                                                                      |
| `key_name`                       | string | 任意 | SSHキーペアの名前                                                                                                                                                                           |
| `user_data`                      | string | 任意 | Base64エンコードされたスタートアップスクリプト（padded Base64、エンコード後 ≤65535バイト・デコード後 ≤49149バイト）。**encode_base64 ツールでエンコードした値を使用。自前エンコード禁止。** |

**adminPass のバリデーション**:

- 9-70文字
- 英大文字、英小文字、数字、記号をそれぞれ1文字以上含む
- 利用可能な記号: `\^$+-*/|()[]{}.,?!_=&@~%#:;'"`
- 正規表現: `^(?=.*[A-Z])(?=.*[a-z])(?=.*[0-9])(?=.*[\\^$+\-*/|()[\]{}.,?!_=&@~%#:;'"])[A-Za-z0-9\\^$+\-*/|()[\]{}.,?!_=&@~%#:;'"]{9,70}$`

---

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

### CreateVolumeRequest（path: `/volumes`）

```json
{
	"volume": {
		"size": 30,
		"description": "<string>",
		"name": "<string>",
		"volume_type": "<string>",
		"imageRef": "<string>"
	}
}
```

| フィールド    | 型             | 必須 | 説明                                                                                                                     |
| ------------- | -------------- | ---- | ------------------------------------------------------------------------------------------------------------------------ |
| `size`        | number         | 必須 | ボリュームサイズ（GB単位）の正の整数。利用可能なサイズは `/types` で取得したボリュームタイプと ConoHa API 側の制約に従う |
| `description` | string \| null | 任意 | ボリュームの説明（nullable）                                                                                             |
| `name`        | string         | 必須 | ボリューム名（1-255文字、英数字・`_`・`-`のみ、正規表現: `^[A-Za-z0-9_-]{1,255}$`）                                      |
| `volume_type` | string         | 必須 | ボリュームタイプ（名前またはID）。事前に `/types` で取得して指定する                                                     |
| `imageRef`    | string         | 任意 | イメージID。**ブートボリューム作成時には必須**                                                                           |

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

### OperateServerRequest（path: `/action`, param: サーバーID）

7つのバリアントから1つを選択:

| バリアント   | requestBody                                | 説明                                |
| ------------ | ------------------------------------------ | ----------------------------------- |
| 起動         | `{"os-start": null}`                       | サーバーを起動する                  |
| 停止         | `{"os-stop": null}`                        | サーバーを停止する                  |
| 強制停止     | `{"os-stop": {"force_shutdown": true}}`    | サーバーを強制シャットダウンする    |
| 再起動       | `{"reboot": {"type": "SOFT" \| "HARD"}}`   | サーバーを再起動する（SOFT / HARD） |
| リサイズ     | `{"resize": {"flavorRef": "<Flavor ID>"}}` | サーバーをリサイズする              |
| リサイズ確定 | `{"confirmResize": null}`                  | リサイズを確定する                  |
| リサイズ取消 | `{"revertResize": null}`                   | リサイズを取り消す                  |

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
