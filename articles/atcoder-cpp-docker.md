---
title: "Dockerを利用したローカルのAtCoder環境（C++）"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["atcoder", "docker", "cpp", "vscode"]
published: true
---

## はじめに

少し前にインストールしている（おそらく）gccのアップデートのせいでAtCoderのコンテスト直前にコンパイルが通らないことに気づき、とても焦るという経験をしました。1年以内にPCを買い替えようとも考えているため、この気に簡単に移行できるDockerを使ったC++のAtCoder環境を作成することにしました。

初学者の方もこの環境を利用すれば、気軽にAtCoderを始められると思います。

Docker環境、及びそのDocker環境を利用したサンプル環境は以下のレポジトリとなります。サンプル環境には検証用に問題がダウンロードされており、一部の問題は解答が書かれています。必要があれば削除してください。

https://github.com/maki8maki/atcoder-cpp-docker

https://github.com/maki8maki/atcoder-cpp-sample

## 実行環境

Macbook Air Intel, macOS Sonoma 14.5
WindowsやLinuxでも同様に実行できると思います。ただし、ショートカットキーは上記の環境でのものを記載しているので、適宜読み替えてください。

## 使い方

まず、[atcoder-cpp-sample](https://github.com/maki8maki/atcoder-cpp-sample)のレポジトリの右上の 'Use this template' から自分用のレポジトリを作成します。続いて、そのレポジトリをcloneしてVSCodeで開きます。さらにVSCodeで `Remote-Containers: Reopen in Container` を選択してコンテナの作成や接続をします。

以上の手順でAtCoder用の環境が整います。

### サンプル入出力のダウンロードとコード生成

`atcoder-tools gen {contest_id}` を実行することでコンテスト環境が `stc/{contest_id}` 以下にダウンロードされます。contest_idはabc357などのように指定します。

### ローカルテスト・サブミット

`Cmd + Shift + P` でコマンドパレットを開き、タスクの実行 → AtCoder Testを選択するとローカルテストが、AtCoder Submitを選択するとサブミットが開いているファイルについて行われます。

### 単独実行

`Ctrl + option + N` で開いているファイルのコンパイルと実行ができるので、デバッグ等に利用できます。

## Docker環境

Dockerfileは以下のようになっています。Dockerfileから作成したイメージは `ghcr.io/maki8maki/zenn-docker` で公開しているため、単独で利用したい場合はこれをプルしてください。

https://github.com/maki8maki/atcoder-cpp-docker/blob/main/Dockerfile

gcc・g++・ac-library・atcoder-toolsなどをインストールしています。環境変数CPLUS_INCLUDE_PATHにac-libraryをダウンロードしたpathを追加しているため、特別な操作をすることなくこのライブラリを使用したファイルのコンパイルが可能です。ac-library・atcoder-toolsについては後で述べます。

atcoder-toolsをインストールするために必要なため、Pythonが利用できる公式イメージをベースとしています。そのため、必要なパッケージを追加でインストールすれば、Python環境にも転用できます。

23行目のコメントでも述べているようにatcoder-toolsでのエラーを防ぐためにmarkupsafeのバージョンを指定してインストールしています。詳細は[この記事](https://zenn.dev/kinakomochi5250/articles/atcoder-tools-error)に書いています。

## AtCoder環境

上記のDocker環境、VSCodeのdevcontainerを利用します。使い方は[ここ](#使い方)に書いてある通りです。拡張機能は最低限のものしか追加していません。

### [ac-library](https://github.com/atcoder/ac-library)

様々なアルゴリズムをAtCoder側で実装したものです。どのようなアルゴリズムが含まれているか、どのように使用するかなどの詳しい内容はGitHubのドキュメントを参照してください。サンプル環境ではatcoder-toolsによるコード生成によってデフォルトで `#include <atcoder/all>` と `using namespace atcoder;` が書き込まれます。例えば、modintの機能を使いたい場合は以下のようになります。

```cpp
#include <atcoder/all>
using namespace atcoder;
using ll = long long;

const ll MOD = 998244353;
using mint = static_modint<MOD>;

mint r = 1;
```

### [atcoder-tools](https://github.com/kyuridenamida/atcoder-tools)

テンプレートからのコード生成やサンプル入出力のダウンロード、ローカルテスト、サブミットを行えるツールです。

ローカルテストとサブミットをターミナルから行う場合は、ファイルがあるディレクトリに移動したり、引数で指定したりしなければならないため、タスクで実行できるようにしています。ショートカットキーを割り当てる場合はkeybindings.jsonに以下のように追加してください。ただし、ショートカットキーはユーザー全体で共通なため、他のものと被らないようにしましょう。

```json

[
    {
        "key": "ctrl+t",
        "command": "workbench.action.tasks.runTask",
        "when": "editorTextFocus && editorLangId == cpp",
        "args": "AtCoder Test"
    },
    {
        "key": "ctrl+s",
        "command": "workbench.action.tasks.runTask",
        "when": "editorTextFocus && editorLangId == cpp",
        "args": "AtCoder Submit"
    },
]

```

コード生成の設定はmy_template.cppやuniversal_code_generator.tomlに書かれています。主にmy_template.cppを変更することでマクロの追加などができます。

### Cookie.txtについて

atcoder-toolsで一度ログインするとCookie情報が保存され、2回目以降はユーザ名やパスワードを入力しなくても良くなります。しかし、コンテナ内に保存されるため、リビルドなどをすると再度入力する必要が出てきます。以下の手順でそれを回避できます。

- [使い方](#使い方)で述べた方法に従ってコンテナの作成・接続をする
- `atcoder-tools gen {contest_id}` などのコマンドでログインする
- `/root/.local/share/atcoder-tools/cookie.txt` を自身のディレクトリにコピーしてくる
- devcontainer.jsonのmountsを以下のように書き換え、リビルドする

```json

"mounts": [
    "source=${localWorkspaceFolder}/cookie.txt,target=/root/.local/share/atcoder-tools/cookie.txt,type=bind", // この行を足す
    "source=${localWorkspaceFolder}/.atcodertools.toml,target=/root/.atcodertools.toml,type=bind"
],

```

Cookie情報があればおそらく他人でもログインできるようになるため、取り扱いに注意してください。GitHubなどにアップロードしてしまった場合はパスワードを変更したほうがいいと思います。サンプル環境では.gitignoreに `cookie.txt` を追加しています。

## 終わりに

Dockerを利用したC++のAtCoder環境を作成しました。自分もあまり使ってないので、不具合があればその都度修正していきたいと思います。また、AHCの入力ジェネレータ・ビジュアライザのツールにもいずれ対応したいです。

## 参考

- [ac-library](https://github.com/atcoder/ac-library)
- [atcoder-tools](https://github.com/kyuridenamida/atcoder-tools)
- [DockerでAtCoderができる環境を作る【Python・C++】](https://qiita.com/hinamimi/items/b3dd159f956628cebdbb)
- [DockerのPython公式イメージにおけるタグの指定について](https://qiita.com/c0a21030/items/6f88c249955bd2ad7ed7)
- [解決策。ImportError: cannot import name 'soft_unicode' from 'markupsafe'](https://keep-loving-python.hatenablog.com/entry/2022/08/29/121515)
