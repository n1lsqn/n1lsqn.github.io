---
title: misskeyのdbをバックアップから復元する。
date: 2024-02-27 14:13:21
tags: misskey
---

# pg_dumpでバックアップを取得し、psqlで復元する。
すでにバックアップは準備できているものとする。
```sh
sudo -u postgres psql
```
```sql
CREATE DATABASE mk1 OWNER misskey;
```

拡張機能を入れている場合は、拡張機能をインストールする。
```sql
CREATE EXTENSION pgroonga;
```

# バックアップから復元する。
```sh
psql -d mk1 < backup.sql
```