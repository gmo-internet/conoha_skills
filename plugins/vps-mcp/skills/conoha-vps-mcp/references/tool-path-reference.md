# ツール別パス・パラメータ一覧

## conoha_get（10パス）

入力: `{ path }`

| パス                         | 概要                               | レスポンス                                      |
| ---------------------------- | ---------------------------------- | ----------------------------------------------- |
| `/servers/detail`            | サーバー一覧取得                   | servers配列（id, name, status, addresses等）    |
| `/flavors/detail`            | フレーバー一覧取得                 | flavors配列（id, name, ram, vcpus, disk等）     |
| `/os-keypairs`               | SSHキーペア一覧取得                | keypairs配列（name, public_key, fingerprint等） |
| `/types`                     | ボリュームタイプ一覧取得           | volume_types配列（id, name等）                  |
| `/volumes/detail`            | ボリューム一覧取得                 | volumes配列（id, name, size, status等）         |
| `/v2/images?limit=200`       | イメージ一覧取得                   | images配列（id, name, status, min_disk等）      |
| `/v2.0/security-groups`      | セキュリティグループ一覧取得       | security_groups配列                             |
| `/v2.0/security-group-rules` | セキュリティグループルール一覧取得 | security_group_rules配列                        |
| `/v2.0/ports`                | ポート一覧取得                     | ports配列（id, device_id, security_groups等）   |
| `/startup-scripts`           | スタートアップスクリプト一覧取得   | スクリプト一覧（name, url等）                   |

**注意**:

- `conoha_get` のパス入力は上記固定文字列のみ受け付ける

---

## conoha_get_by_param（6パス）

入力: `{ path, param }`

| パス                         | param                        | 概要                                         |
| ---------------------------- | ---------------------------- | -------------------------------------------- |
| `/ips`                       | サーバーID                   | サーバーに紐づくIPアドレス一覧取得           |
| `/os-security-groups`        | サーバーID                   | サーバーに紐づくセキュリティグループ一覧取得 |
| `/rrd/cpu`                   | サーバーID                   | サーバーのCPU使用率統計取得                  |
| `/rrd/disk`                  | サーバーID                   | サーバーのディスク使用率統計取得             |
| `/v2.0/security-groups`      | セキュリティグループID       | セキュリティグループ詳細取得                 |
| `/v2.0/security-group-rules` | セキュリティグループルールID | セキュリティグループルール詳細取得           |

---

## conoha_post（5パス）

入力: `{ input: { path, requestBody } }`

| パス                         | 概要                           | requestBody のルートキー |
| ---------------------------- | ------------------------------ | ------------------------ |
| `/servers`                   | サーバー作成                   | `server`                 |
| `/os-keypairs`               | SSHキーペア作成                | `keypair`                |
| `/volumes`                   | ボリューム作成                 | `volume`                 |
| `/v2.0/security-groups`      | セキュリティグループ作成       | `security_group`         |
| `/v2.0/security-group-rules` | セキュリティグループルール作成 | `security_group_rule`    |

requestBody の詳細は [request-body-schemas.md](request-body-schemas.md) を参照。

---

## conoha_post_put_by_param（6パス）

入力: `{ input: { path, param, requestBody } }`

| パス                     | param                  | 概要                                             |
| ------------------------ | ---------------------- | ------------------------------------------------ |
| `/action`                | サーバーID             | サーバー操作（起動/停止/再起動/リサイズ等）      |
| `/remote-consoles`       | サーバーID             | リモートコンソールURL取得                        |
| `/os-volume_attachments` | サーバーID             | ボリュームアタッチ                               |
| `/v2.0/security-groups`  | セキュリティグループID | セキュリティグループ更新                         |
| `/v2.0/ports`            | ポートID               | ポート更新（セキュリティグループの関連付け変更） |
| `/volumes`               | ボリュームID           | ボリューム更新                                   |

requestBody の詳細は [request-body-schemas.md](request-body-schemas.md) を参照。

---

## conoha_delete_by_param（5パス）

入力: `{ path, param, confirm }`

| パス                         | param                        | 概要                           |
| ---------------------------- | ---------------------------- | ------------------------------ |
| `/servers`                   | サーバーID                   | サーバー削除                   |
| `/os-keypairs`               | SSHキーペア名                | SSHキーペア削除                |
| `/v2.0/security-groups`      | セキュリティグループID       | セキュリティグループ削除       |
| `/v2.0/security-group-rules` | セキュリティグループルールID | セキュリティグループルール削除 |
| `/volumes`                   | ボリュームID                 | ボリューム削除                 |

**注意**:

- 削除操作は元に戻せないため、必ず `confirm: true` を指定する
