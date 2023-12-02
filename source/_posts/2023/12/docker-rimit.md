---
title: dockerにはコンテナ当たりの容量制限があるという話。
date: 2023-12-02 23:13:33
tags: docker
thumbnailImage: https://www.docker.com/wp-content/uploads/2023/08/logo-guide-logos-1.svg
---

知らなかったのだが、Dockerにはコンテナ当たりの容量制限があるらしい。
経緯としては、ストレージがあるのにもかかわらず、no space left on deviceというエラーが出てしまうというもの。

<!-- more -->
<!-- toc -->

[Local Development (Windows 10)
](https://unrealcontainers.com/docs/environments/local-windows-10#:~:text=By%20default%2C%20Docker%20Desktop%20for,by%20following%20the%20instructions%20below)

>By default, Docker Desktop for Windows imposes a 20GB size limit on container images, which is too low for building and running Unreal Engine containers. You will need to increase the maximum container disk size to the recommended limit of 300GB by following the instructions below.

### 設定
Docker Desktopの設定から、Docker Engineの設定を開き、`"defaultKeepStorage": "20GB",`を`"defaultKeepStorage": "200GB",`に変更する。（各々好きな値で）

おしまい。