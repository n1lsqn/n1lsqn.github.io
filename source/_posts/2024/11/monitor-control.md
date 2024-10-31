---
title: SteamVRを起動したらモニターの接続を解除してVRChatのパフォーマンスを改善する
date: 2024-11-01 07:11:30
tags:
  - SteamVR
  - VRChar
thumbnailImage: source/_posts/2024/11/monitor-control/VRChat_2024-10-28_23-13-55.719_1920x1080.png
---

最近VR機器を使うことが増えてきており、マルチモニター環境においてSteamVRやVRChat等のVRゲームを起動したときに、モニターの接続を解除してあげるとFPSが自環境では5-10FPSの向上が見られたので備忘録。

<!-- more -->
<!-- toc -->

# multimonitortoolの準備
[multimonitortool](https://www.nirsoft.net/utils/multi_monitor_tool.html)をダウンロード。結構下のほうにあるのでCtrl+FでDownloadと検索したほうが幸せかもしれない。Languageにある日本語化ファイルもダウンロードしておく。

# multimonitortoolでモニターIDの確認
{% asset_img {2275EDBB-9E33-4D0F-945F-E54DA39CCF39}.png %}

起動して、モニターidを取得する。
下部分に表示されているウィンドウと照らし合わせながら、モニターを右クリック

{% asset_img {82CD15FE-57B8-424B-8C19-6DD341C76DF6}.png %}

モニターIDをメモしておく

# batファイルの作成

off時のbat
``` bat
@echo off
setlocal

:: monitor IDs
set "MAIN_MONITOR_ID=MONITOR\*******\{*******-****-****-****-************}\****"
set "SUB1_MONITOR_ID=MONITOR\*******\{*******-****-****-****-************}\****"
set "SUB2_MONITOR_ID=MONITOR\*******\{*******-****-****-****-************}\****"

:: MultiMonitorToolで切断
"MultiMonitorTool.exe" /disable "%SUB1_MONITOR_ID%"
"MultiMonitorTool.exe" /disable "%SUB2_MONITOR_ID%"

:: 自動的に閉じる
timeout /t 3 >nul
```

on時のbat
``` bat
@echo off
setlocal

:: Define monitor IDs
set "MAIN_MONITOR_ID=MONITOR\*******\{*******-****-****-****-************}\****"
set "SUB1_MONITOR_ID=MONITOR\*******\{*******-****-****-****-************}\****"
set "SUB2_MONITOR_ID=MONITOR\*******\{*******-****-****-****-************}\****"

:: MultiMonitorToolで切断
"MultiMonitorTool.exe" /enable "%SUB1_MONITOR_ID%"
"MultiMonitorTool.exe" /enable "%SUB2_MONITOR_ID%"

:: 自動的に閉じる
timeout /t 3 >nul
```

まず`set MAIN_MONITOR_ID`、`set SUB1_MONITOR_ID`、`set SUB2_MONITOR_ID`でモニターIDを定義、私はトリプル環境のため三つ定義してあるため、各々そこは変更していただきたい。

`"MultiMonitorTool.exe"`はMultiMonitorTool.exeが配置されている絶対パス(エクスプローラーでexeを指定し右クリック -> パスのコピーで取得)を指定

これでとりあえず接続/切断は使えるはず。

# autohotkeyの構築
[autohotkey](https://www.autohotkey.com/)を使ってSteamVR`vrserver.exe`が起動したときに切断、終了時に接続というようなことをする。ダウンロードしてインストールしておく。

``` ahk
#Persistent
SetTimer, CheckSteamVR, 1000
steamVRStarted := false
return

CheckSteamVR:
Process, Exist, vrserver.exe
if (ErrorLevel != 0 and !steamVRStarted) {
    ; SteamVR起動時に実行
    Run, "off.bat"
    steamVRStarted := true
}
else if (ErrorLevel == 0 and steamVRStarted) {
    ; SteamVR終了時に実行
    Run, "on.bat"
    steamVRStarted := false
}
return
```
`off.bat`と`on.bat`は先ほどのパスをコピーから取得する

先ほど書いたahkファイルを開くと

{% asset_img {C695ED36-8484-413B-809D-B34F37F620CB}.png %} 

といわれるのではいを選択、now installみたいなウィンドウが出てきたらok。
またahkファイルを開いてシステムトレイに追加されてたらok。緑でHと書いてあるロゴがそれ

{% asset_img {73A5AC60-1C32-4DA3-8954-42F0548F1215}.png %}

だいぶ駆け足気味で書いているので、わかんなかったら遠慮なくTwitterのDMまでぜひ

# 参考
[WSL2でマルチディスプレイの接続/切断をCLIで行う](https://note.com/ngc_shj/n/n257e2c5b991c)