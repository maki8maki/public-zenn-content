---
title: "DockerとVSCodeを用いたZenn執筆環境"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "docker", "vscode"]
published: true
---

## はじめに

これまでQiitaやZennを利用するのみでしたが、これからは自分も執筆する側になりたいと思い、記事の投稿を始めました。
1年以内にPCを買い替える予定のため、Dockerを利用したZenn CLI環境を用意しました。

Docker環境のレポジトリは以下となっています。

https://github.com/maki8maki/zenn-docker

また、サンプルの執筆環境のレポジトリは以下となっています
執筆環境はGitHubと連携することを前提としています。

https://github.com/maki8maki/zenn-content-sample

## Docker

Dockerfileは以下のようになっていて、node-alpineにZenn CLIをnpmでインストールしたものとなっています。

https://github.com/maki8maki/zenn-docker/blob/main/Dockerfile

上記のイメージは `ghcr.io/maki8maki/zenn-docker` で公開しているため、単独で利用したい場合はこれをプルしてください。

## 執筆環境

### 基本的な使い方

VSCodeのDev Containerを利用してコンテナを作成し、その中でZenn CLIによる記事の執筆をします。
Dev Containerの使い方の詳細はここでは省略させていただきます。

普段はmainブランチとは別のブランチで作業し、mainブランチにPRすることを想定しています。

拡張機能のZenn EditorによるVSCode上でのプレビューが可能です。
`npx zenn preview` を実行した後に、[http://localhost:8000](http://localhost:8000) にアクセスすることでブラウザ上でもプレビューを確認できます。

### 画像の貼り付け

VSCodeでファイルを開いて `Ctrl` + `V` とすると、クリップボードにコピーした画像を以下のようなマークダウン記法で挿入できます。
画像は `images/{articles or books}/{slug}/` 以下に保存されます。

``` markdown
![alt text](../images/articles/sample-20240528/image.png)
```

ただし、Zenn CLIでは `/images/` から始まる絶対パスを指定する必要があるので、以下のように修正する必要があります。

``` markdown
![alt text](/images/articles/sample-20240528/image.png)
```

### 校正

mainへのPR時にtextlintによる校正が行われます。
また、ローカルでも拡張機能のvscode-textlintを利用して文章の校正が可能です。

校正の設定は `.textlintrc` に記述されています。

textlintによる指摘点の中には機械的に修正可能なものがあります。
VSCodeのターミナルで `npm run fix` を実行することでこれらの修正が行われます。
修正の対象となるのはorgin/mainブランチとの差分があるファイルです。

### 拡張機能

`devcontainer.json` に記載されているデフォルトでダウンロードされる拡張機能について説明します。

#### Zenn Editor

Zenn CLIの利用を簡単にするための拡張機能です。

`Ctrl` + `Shift` + `P` 後に `Zenn Editor: Open Tree View Item` を選択すると、エクスプローラーにZENN CONTENTSが表示されます。
ZENN CONTENTSにはarticlesとbooksごとにタイトルの一覧が表示され、該当する記事のマークダウンファイルを開くことができます。
記事を開くと、右上には通常のプレビューボタンに加え、Zenn Editorで実装されているプレビューボタンも表示されます。
ZENN CONTENTSの文字の脇にはarticles、booksの作成ボタンもあり、ワンクリックで新規の記事の作成が可能です。

#### Git Graph

Gitリポジトリの履歴やブランチを視覚化するためのGUIツールです。
マージなどのコマンドも実行可能です。

#### Markdown All in One

マークダウンを書く際に便利なショートカット等が含まれています。

#### markdownlint

見出しの後は1行あけるなどのマークダウンのスタイルチェックを行っています。
ルールが比較的厳しいとされており、好き嫌いは個人で分かれます。
ZennではGitHubなどのコンテンツの埋め込みにはURLを直接貼り付けますが、markdownlintではこれを禁止するルールが存在します。
そのため、このルールを適用しないように除外しています。
その他のルールについても `setting.json` に追加することで除外できます。

#### vscode-textlint

textlintをローカルで行うための拡張機能です。
保存時に実行する設定となっています。

#### Trailling Spaces

行末のスペースを除外してくれます。
マークダウンで入り込むことはほとんどないですが、念の為追加しています。

## 最後に

Docker環境とそれを利用してVSCodeでZennを執筆するサンプル環境を作成しました。

## 参考

* [Zenn CLI環境をRemote Containersを利用して構築](https://zenn.dev/sg4k0/articles/906d97bfeb5aef)
* [Zenn CLI docker環境で簡単構築](https://qiita.com/CopyAndPaste/items/6c04950d9fe57c6cfe76)
