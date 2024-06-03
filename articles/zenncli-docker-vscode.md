---
title: "DockerとVSCodeを用いたZenn執筆環境"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "docker", "vscode"]
published: false
---

これまでQiitaやZennを利用するのみでしたが、これからは自分も執筆する側になりたいと思い、記事の投稿を始めました。
1年以内にPCを買い替える予定のため、Dockerを利用したZenn CLI環境を用意しました。

Docker、サンプルの執筆環境のレポジトリは以下となっています。

https://github.com/maki8maki/zenn-docker


https://github.com/maki8maki/zenn-content-sample

## Docker

Dockerfileは以下のようになっていて、node-alpineにZenn CLIをnpmでインストールしたものとなっています。

https://github.com/maki8maki/zenn-docker/blob/main/Dockerfile
