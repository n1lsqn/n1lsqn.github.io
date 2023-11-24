---
title: Misskeyでハイライトやサーバークラウドのアイコンがおかしい問題の対処法
date: 2023-11-20 01:28:06
tags:
  - misskey
  - redis
thumbnailImage: https://raw.githubusercontent.com/misskey-dev/assets/main/banner.png
---

#### 問題点
- ハイライトが表示されない
- サーバークラウドのアイコンが表示されない

<!-- more -->
<!-- toc -->

#### 原因
- Redisが古いことが原因
不確かだが、v7以上が必須のはずがv6が入っていた

#### 対処法
- Redisを最新版にアップデートする

``` bash terminal
sudo apt install -y curl ca-certificates gnupg2 lsb-release

curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt update

sudo apt install -y redis
```

これでok。ついでにPostgreSQLも古いバージョンを利用していたためアップデートした

#### PostgresSQLのアップデート
**必ずpg_dump等でバックアップを取ってから行うこと**
``` bash terminal
sudo apt install postgresql-15
sudo pg_dropcluster 15 main --stop
sudo pg_upgradecluster -v 15 14 main
```
`pg_updatecluster`で自動的にアップデートされ切り替わるので少しビビった

#### 参考
- [Ubuntu版Misskeyインストール方法詳説](https://misskey-hub.net/docs/install/ubuntu-manual.html#%E3%81%9D%E3%81%AE%E4%BB%96%E3%81%AEmisskey%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E6%96%B9%E6%B3%95)
- [Ubuntu 22.04 LTS で Postgresql 13から15にアップグレードした話(pg_upgradecluster編)](https://qiita.com/ynott/items/ca130c4b70533a91d3d3)