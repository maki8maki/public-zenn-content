---
title: "X11 ForwardingによるMacでのGUI表示"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ssh", "x11", "mac", "ubuntu"]
published: false
---

## はじめに

自分で作業をするときに以下のQiitaの記事の情報が古くなっていることに気づいたので、内容をアップデートしてみました。

https://qiita.com/loftkun/items/37340745f211ea5d7ece

## サーバでの準備

ここではUbuntuでの例を上げます。

sshdの設定ファイルを変更し、sshdを再起動します。

```bash
sudo vim /etc/ssh/sshd_config
# 以下の設定を行う、（自分の環境では一部はデフォルトで設定されていた）
# X11Forwarding yes
# X11DisplayOffset 10
# X11UseLocalhost no
sudo systemctl restart sshd
```

## Macでの準備

### XQuartzのインストール

インストールは以下のコマンドを実行するか、[Xquartz-x.x.x.pkg](https://www.xquartz.org/)をダウンロードして行います。

```bash
brew install --cask xquartz
```

以下のコマンドを実行し、環境変数が設定されていることを確認します。

```bash
echo $DISPLAY
# /private/tmp/com.apple.launchd.xxxxxxxxxx/org.xquartz:0
```

### ssh接続

以上の準備を終えるとGUIの表示ができるようになります。`-XY` オプションをつけてssh接続し、GUIを表示するコマンドを実行します。

```bash
ssh -XY my-server

# サーバでの操作例
xeyes
# 目玉の表示がされたら成功
```

`.ssh/config` ファイルで以下のように設定すると、`-XY` オプションが不要になります。

```config
Host my-server
    HostName xxx.xx.xxx.xx
    User hoge
    ForwardX11 yes
    ForwardX11Trusted yes
```
