---
title: "tmuxの使い方"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["tmux"]
published: false
---

## はじめに

自分の備忘録も兼ねて、tmuxの基本的なコマンドの使い方等をまとめています。

## tmuxとは

- 端末多重化ソフトウェア（Terminal Multiplexer）の略
- 仮想端末がセッションで管理されているため接続端末と通信が切れても作業を復旧可能、ターミナルを終了してもセッションは維持される
  - 外部からssh接続して時間がかかる処理を回しても接続を維持する必要がない
- セッション、ウィンドウ、ペインから成る
  - ひとつのターミナルは1つ以上のセッションから成る
  - ひとつのセッションは1つ以上のウィンドウから成る
  - ひとつのウィンドウは1つ以上のペインから成る

## 基本的なコマンド操作

tmuxではプレフィックスキーと特定のキーを順に押下する操作があります。プレフィックスキーはデフォルトで `Ctrl+b` です。以下ではプレフィックスキーを "PK" と書きます。

### セッション操作

#### セッションを起動する

```bash
tmux
```

#### 名前付きでセッションを起動する

```bash
tmux new -s <name>
```

#### セッションを一覧表示する

```bash
tmux ls
tmux list-sessions
```

または `PK + s`

一覧からそれぞれのセッションに移動できます。

#### セッションから離れる（Detach）

`PK + d`

#### 特定のセッションに再接続する（Attach）

```bash
tmux a -t <name>
```

#### セッションを終了する

```bash
exit
```

#### 特定のセッションを終了する

```bash
tmux kill-session -t <name>
```

### ウィンドウ操作

#### ウィンドウを作成する（Create）

`PK + c`

#### 次のウィンドウに移動する（Next）

`PK + n`

#### 前のウィンドウに移動する（Previous）

`PK + p`

#### 数字で指定したウィンドウに移動する

`PK + 数字`

#### ウィンドウを一覧表示する（Window）

`PK + w`

一覧からそれぞれのセッションやウィンドウに移動できます。

#### ウィンドウを終了する（確認メッセージあり）

`PK + &`

#### ウィンドウを終了する

```bash
exit
```

### ペイン操作

#### ペインを左右に分割する

```bash
tmux splitw -h
```

または `PK + %`

#### ペインを上下に分割する

```bash
tmux splitw -v
```

または `PK + "`

#### ペインを上下左右に移動する

```bash
tmux selectp -U
tmux selectp -D
tmux selectp -L
tmux selectp -R
```

または `PK + 矢印`

`selectp` は `select-pane` でも可能です。

#### 次のペインに移動する

`PK + o`

#### 前のペインに移動する

`PK + ;`

#### 時刻を表示する

`PK + t`

`Esc` や `q` で終了できます。

#### ペインを終了する（確認メッセージあり）

`PK + x`

#### ペインを終了する

```bash
exit
```

### コピー&ペースト

viモードで使用するのが様々な場所で推奨されているため、その方法を記載しています。viモードの設定方法は[カスタマイズの章](#カスタマイズ)で紹介しています。

1. コピーモードに移る：`PK + [`
   - コピーモードを終了する：`q`
2. コピーする先頭の位置にカーソルを移動する
3. コピーを開始する：`v`
   - 選択の解除：`Esc`
4. コピーする末尾の位置にカーソルを移動する
5. コピーをしてコピーモードを終了する：`Enter`
6. コピーしたものを貼り付ける：`PK + ]`

`V` で行選択も可能です。

## カスタマイズ

~/.tmux.confファイルで設定のカスタマイズが可能です。以下に基本的な設定を示します。

```bash:.tmux.conf
# プレフィックスキーを Ctrl+b から Ctrl+aに変更する
set -g prefix C-a
unbind C-b

# マウス操作をオンにする
set-option -g mouse on

# コピーモードのキー操作をviのようにする
set-window-option -g mode-keys vi
bind -T copy-mode-vi v send-keys -X begin-selection
bind -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel
bind -T copy-mode-vi V send-keys -X select-line
```

## 参考

- [便利なtmuxの使い方をまとめてみる](https://zenn.dev/azunasu/articles/25d9999ca0fb96)
- [アニメーションで学ぶtmux入門 ～精選10機能～](https://qiita.com/KoyanagiHitoshi/items/318d4b8ef3b4e5b87390)
- [とほほのtmux入門](https://www.tohoho-web.com/ex/tmux.html)
- [tmux（2.6以降）のコピーモードについてまとめた（for vimmer）](https://qiita.com/shimmer22/items/67ba93060ae456aadd1b)
