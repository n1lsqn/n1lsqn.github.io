---
title: Misskey 2023.11.0にmeilisearchを導入する
date: 2023-11-20 01:27:06
tags: 
  - misskey
  - meilisearch
---

### Misskeyにmeilisearchを導入する方法

<!-- more -->
<!-- toc -->

読み飛ばしたい人は6まで飛ばすことをお勧めする
#### meilisearchのインストール
[公式ドキュメント](https://www.meilisearch.com/docs/learn/cookbooks/running_production)をもとに進めていく

``` bash terminal
sudo apt update
sudo apt install curl -y
curl -L https://install.meilisearch.com | sh
chmod +x meilisearch
sudo mv ./meilisearch /usr/local/bin/
sudo useradd -d /var/lib/meilisearch -b /bin/false -m -r meilisearch
```
#### 設定ファイルの書き換え
``` bash terminal
curl https://raw.githubusercontent.com/meilisearch/meilisearch/latest/config.toml > meilisearch.toml
nano meilisearch.toml
```
master_keyは16バイト以上に必要らしいので、以下のコマンドで生成する
``` bash terminal
sudo apt install uuid
uuid
```

``` toml meilisearch.toml
env = "production"
master_key = "YOUR_MASTER_KEY_VALUE"
db_path = "/var/lib/meilisearch/data"
dump_dir = "/var/lib/meilisearch/dumps"
snapshot_dir = "/var/lib/meilisearch/snapshots"
```
以上の点を書き換え後、`sudo mv meilisearch.toml /etc/meilisearch/`で設定ファイルを移動する

移動後、ディレクトリを生成する
``` bash terminal
sudo mkdir /var/lib/meilisearch/data /var/lib/meilisearch/dumps /var/lib/meilisearch/snapshots
sudo chown -R meilisearch:meilisearch /var/lib/meilisearch
sudo chmod 750 /var/lib/meilisearch
```

#### サービスの起動
`sudo nano /etc/systemd/system/meilisearch.service`
``` systemd /etc/systemd/system/meilisearch.service
[Unit]
Description=Meilisearch
After=systemd-user-sessions.service

[Service]
Type=simple
WorkingDirectory=/var/lib/meilisearch
ExecStart=/usr/local/bin/meilisearch --config-file-path /etc/meilisearch.toml
User=meilisearch
Group=meilisearch

[Install]
WantedBy=multi-user.target
```

``` bash terminal
sudo systemctl enable meilisearch
sudo systemctl start meilisearch
sudo systemctl status meilisearch
```

#### APIキーの取得
``` bash terminal
curl \
  -X GET 'http://localhost:7700/keys' \
  -H 'Authorization: Bearer MASTER_KEY'
```
`MASTER_KEY`にはmeilisearch.tomlで設定したmaster_keyを入れる

次のように返ってくるため
``` json
{
  "results": [
    {
      "name": "Default Search API Key",
      "description": "Use it to search from the frontend",
      "key": "d0552b41536279a0ad88bd595327b96f01176a60c2243e906c52ac02375f9bc4",
      "uid":"74c9c733-3368-4738-bbe5-1d18a5fecb37",
      "actions": [
        "search"
      ],
      "indexes": [
        "*"
      ],
      "expiresAt": null,
      "createdAt": "2022-01-01T10:00:00Z",
      "updatedAt": "2022-01-01T10:00:00Z"
    },
    {
      "name": "Default Admin API Key",
      "description": "Use it for all other than search operations. Caution! Do not expose it on a public frontend",
      "key": "380689dd379232519a54d15935750cc7625620a2ea2fc06907cb40ba5b421b6f",
      "uid": "20f7e4c4-612c-4dd1-b783-7934cc038213",
      "actions": [
        "*"
      ],
      "indexes": [
        "*"
      ],
      "expiresAt": null,
      "createdAt": "2021-08-11T10:00:00Z",
      "updatedAt": "2021-08-11T10:00:00Z"
    },
    {
      "name": null,
      "description": "Search patient records key",
      "key": "d0552b41536279a0ad88bd595327b96f01176a60c2243e906c52ac02375f9bc4",
      "uid": "ac5cd97d-5a4b-4226-a868-2d0eb6d197ab",
      "actions": [
        "search"
      ],
      "indexes": [
        "patient_medical_records"
      ],
      "expiresAt": "2023-01-01T00:00:00Z",
      "createdAt": "2022-01-01T10:00:00Z",
      "updatedAt": "2022-01-01T10:00:00Z"
    }
  ],
  "offset":0,
  "limit":20,
  "total":3
}
```
Default Admin API Keyの`key`をコピーする

#### Misskeyの設定
misskeyの設定ファイルを開く
``` bash terminal
sudo nano /home/misskey/config/default.yaml
```

``` yaml /home/misskey/config/default.yaml
meilisearch:
  host: localhost
  port: 7700
  apiKey: '先ほどコピーしたkeyを入れる'
#  ssl: true
  index: 'misskey'
#  scope: local
```
scopeはlocalにすると、ローカルのみ検索できるようになる
indexはdomain名が推奨されている模様。今回はmisskeyとした

#### indexを張る
ここからが本番といっても過言ではないだろう。

{% mastodon https://fedibird.com/@noellabo/111419037137255692 %}

上記の通り、`2023.11`から`createdAt`が削除されているのだ。

そのため、[misskey_meilisearch_importer](https://github.com/noellabo/misskey_meilisearch_importer)を使ってindexを張る必要がある。

``` bash terminal
cd /home/misskey

git clone https://github.com/noellabo/misskey_meilisearch_importer.git
cd misskey_meilisearch_importer

pnpm install
```

``` bash terminal
node importer.js --config /home/misskey/.config/default.yml
```
これでindexが張られる。これでaid形式のインポートは終了。`misskey -> ダッシュボード -> ロール`よりノートの検索を有効にし、検索ができるようになる。

#### 私の場合
まず、aidxのため、うまくいかなかった。
そのため、forkし、修正したものを使っている。
[misskey_meilisearch_importer](https://github.com/n1lsqn/misskey_meilisearch_importer.git)

#### 参考
[MisskeyでMeilisearchを導入するやり方](https://nanasi-apps.xyz/misskey-meilisearch)
[のえるさんの投稿](https://fedibird.com/@noellabo/111419037137255692)
[公式ドキュメント](https://www.meilisearch.com/docs/learn/cookbooks/running_production)