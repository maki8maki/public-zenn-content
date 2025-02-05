---
title: "LaTeXの使い方"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["latex"]
published: true
---

## はじめに

自分の備忘録も兼ねて、$\LaTeX$の基本的なコマンドの使い方等をまとめています。

簡単な例を[このレポジトリ](https://github.com/maki8maki/latex-example)で紹介しています。

:::message alert
ここに書かれている情報は2021年ごろに調べた情報をZennで改めて整理したものであるため、一部古い内容を含んでいる可能性があります。
:::

## 環境構築

### ローカル

VSCodeを利用したローカルでの環境構築を紹介します。実行環境はmacOS Big Sur 11.5.2です。

:::message
下記の方法で実際に環境構築したのは2021年ぐらいであり、現在でも同じようにできることは保証しない
:::

#### TeXの準備

- MacTeXのインストール
  - `brew install mactex --cask`
  - 相当時間がかかる
  - GUIツールを必要としなければmactex->mactex-no-guiとする

- LaTeXパッケージのアップデートを行う
  - `sudo tlmgr update --self --all`
  - tlmgrコマンドが無いと言われた場合は、.zprofileに `export PATH=/usr/local/texlive/{VERSION}/bin/universal-darwin:$PATH` を追加して再度実行
  - こちらも時間がかかる
- 用紙サイズをA4にする
  - `sudo tlmgr paper a4`
- ビルド方法の設定
  - ~/.latexmkrcに記述する
  - [ここ](https://qiita.com/rainbartown/items/d7718f12d71e688f3573)を参考にすると良い

#### VSCodeの設定

- LaTeX Workshopという拡張機能をインストールする
- setting.jsonに色々と書く
  - [ここ](https://pyteyon.hatenablog.com/entry/2019/12/24/225305#5-2-LaTeX-Workshop-%E3%81%AE%E8%A8%AD%E5%AE%9A%E3%82%92%E8%A6%8B%E7%9B%B4%E3%81%99)を参考にするとよい

#### コード整形

- コードの自動整形ができないとき...
- macの場合は、`/Library/Developer/CommandLineTools/SDKs/` を見て、MacOSX(バージョン).sdkフォルダのバージョン部分がPCのバージョンと一致するか確かめる
- 古いバージョンしかない場合は、`sudo rm -r /Library/Developer/CommandLineTools` で削除後に `xcode-select --install` でインストールし直せば解決することがある

### ローカル(docker)

- ローカルでdocker+vscode
  - ローカル環境を汚さない
- [このレポジトリ](https://github.com/being24/latex-template-ja)が非常に有用である

### オンライン

- 複数台のPCを使う場合はオンラインでのLaTeXエディターを利用すると便利である
- 例：[overleaf](https://ja.overleaf.com/)

## LaTeXを使う

### 基本

#### 文書の設定

- `\documentclass[オプション]{文書クラス}` で文書全体の設定をする

代表的な文書クラスの種類は以下の通りです。

| 文献種類   | 欧文    | 和文     | 和文（新） | 和文（縦書き） |
| :--------: | :-----: | :------: | :--------: | :------------: |
| **論文**   | article | jarticle | jsarticle  | tarticle       |
| **本**     | book    | jbook    | jsbook     | tbook          |
| **報告書** | report  | jreport  |            | treport        |

代表的なオプションの種類は以下の通りです。ただし、文書クラスによって使用できるオプションは異なります。

|オプション|概要|
| -- | -- |
|(n)pt|文字サイズをnポイントにする（デフォルトは10）|
|(size)paper|用紙サイズをsizeにする|
|onecolumn|文章を1段組にする（デフォルト）|
|twocolumn|文章を2段組にする|
|oneside|奇数・偶数ページで同じレイアウト（article, reportでのデフォルト）|
|twoside|奇数・偶数ページで異なるレイアウト（bookでのデフォルト）|
|fleqn|数式を左寄せで出力（デフォルトは中央）|
|leqno|数式番号を左寄せで出力（デフォルトは中央）|
|landscape|用紙の向きを横長にする|

::: message
最近利用が増えているlualatexでは上記の文書クラスをそのままでは利用できない。[ここ](https://qiita.com/MIZOGUCHIKoki/items/d1603d59ccbc08438520)を参考に別のクラスやluatexjaパッケージを利用する。
:::

#### 文章本体

- 文章本体は `\begin{document}〜\end{document}` の間に書く

#### タイトル

- `\title{}}` で題、`\auther{}` で著者を指定し、`\maketitle` で出力する
- 日付けはコンパイル時のものが表示されるが、`\date{}` で指定可能

#### 章立て

- `\section{}` でセクションの見出しを出力でき、番号も自動で割り振られる
- セクションの下位のサブセクションの見出しを出力するには `\subsection{}` を用いる
- 番号が不要なときはsection, subsectionの末尾に*をつける

#### 箇条書き

- 番号を振らない場合は `\begin{itemize} ~ \end{itemize}` の間に書く
- 番号を振る場合は `\begin{enumerate} ~ \end{enumerate}` の間に書く
  - 上2つはそれぞれの行を `\item 内容` と書いていく
- 語句説明のときは `\begin{description} ~ \end{description}` の間に書く
  - 語句説明はそれぞれの行を `\item[語句] 説明` で書いていく

#### 改行

- 段落改行と強制改行がある
  - 前者では次の行で字下げされるが、後者ではされない
- 段落改行をするには1行以上の空行を入れるか、行末に `\par` をつける
- 強制改行をするには行末に `\\` をつけて次の行を空行にしないか、空行を入れた上で次の文章の前に `\noindent` をつける

#### パッケージ

- LaTeXの新たな機能の追加のために用いる
- `\documentclass}` と `\begin{document}` の間（プリアンブル）に `\usepackage[オプション]{パッケージ名}` と書くことで使えるようになる
  - 複数パッケージを用いる際には{}の中にコンマで区切って並べることもできる
- よく用いるパッケージ
  - 数式：amsmath, bm
  - 図表：graphicx, float, subcaption

### 数式

#### 基本

- 行内数式は `$` で挟む
- 数式を独立した行に数式番号付きで書くときは `\begin{equation} ~ \end{equation}` の間に書く
  - 数式番号を書かないときは `\[ ~ ]\` の間に書く
  - amsmathパッケージを用いると `equation*` とすることで番号なしにできる

#### 上付き・下付き文字

- 上付き・下付き文字はそれぞれ `^{n}`、 `_{n}` と書く
  - 1文字のときは、{}はなくてもよい

#### 分数

- 分数は `\frac{分子}{分母}` と書く
- sinの括弧の中などで分数を小さくしたいときは `\tfrac{}{}` を用いる
  - `\tfrac{}{}` はテキストスタイル（通常の文章）で表示する命令なので、ディスプレイスタイル（数式）で用いると相対的に小さくなる

#### 三角関数・対数・根

- `\hoge` でよくプログラムで使うような関数を書ける
  - 三角関数：`\sin`、`\cos`、`\tan`
  - 対数：`\log`
    - 後ろに `_{}` をつけると底が書ける
  - 平方根：`\sqrt{}`
    - n乗根は `\sqrt[n]{}` で表記可能

#### 括弧

- `\left( ... \right)` で中身に応じて大きさが変わる括弧を書ける
- `\left.` とすると左側の括弧を消せる（右側も同様）

#### 記号

- 円周率：`\pi`
- 無限大：`\infty`
- ×：`\times`
- ±：`\pm`
- ÷：`\div`
- ・：`\cdot`
- 点々：`\ldots`、`\vdots}`、`\ddots`
  - それぞれlower（下）、vertical（垂直）、diagonal（対角線）に沿った点線
- 積分記号：`\int`
  - 後ろに `_{min}^{max}` をつけると定積分にできる
- 総和・総乗：`\sum}`、`\prod`
  - 後ろに `_{min}^{max}` をつけると下限・上限を書き込める
- 偏微分：`\partial`

#### 行列

- 行列は `\begin{pmatrix} ~ \end{pmatrix}` を用いて表示する
- 行列式は `\begin{vmatrix} ~ \end{vmatrix}` を用いて表示する
- `&` で行を、`\\` で列を区切る
  - `\\` を用いるにはamsmathパッケージが必要

#### ベクトル

- 太字のベクトルは `\bm` を用いて表示する
  - bmモジュールが必要

#### 場合分け

- `\begin{cases} ~ \end{cases}` の間に書く
- `&` で行を、`\\` で列を区切る
  - `\\` を用いるにはamsmathパッケージが必要

#### スペース

- 数式の中で単にカンマを入れると、行列とみなされて微小な空きが入るので、数字を区切りたいときは `{,}` とすると空きがなくなる
- 数字と単位の間のように微小な空きを入れたい場合は `\,` を入れる

#### 単位

- 単位をローマン体で書くとき（科学論文のルールだとそうなっている、LaTeXの数式中の英字は通常イタリック体）は `\mathrm` コマンドを用いる
- `\mathrm` では、組み立て単位で `\cdot` を用いたときに、前後に余分な余白が入るので、このときは `\text{}` を使う
  - `\text{}` の{}の中身は通常文章の出力なので、`\cdot` や上付き文字の前後などは `$` で挟む
- siunitxパッケージを用いると容易に上記を考慮した書き方ができる
  - 詳しい使い方は[ここ](https://kumaroot.readthedocs.io/ja/latest/latex/latex-siunitx.html)を見る

### 図

#### 基本

- 図を入れるときにはgraphicxパッケージを用いる
- その際にオプションとして `dvipdfmx` を指定する
  - このオプションはdvipdfmxが解釈できるように画像取り込みを行うことを指示するもの
- 図の挿入は `\includegraphics[オプション]{ファイル名}` でできる
  - オプションはコンマ区切りで複数入れることも可能
- 図を中央揃えにしたければ `\begin{center} ~ \end{center}` で挟む
  - また、すぐ下に文字を入れる場合は `\\` で区切って記入する

includegraphicsのオプションは以下の通りです。

|オプション|機能|
| -- | -- |
|keepaspectratio|縦横比を維持する|
|scale|図のサイズの指定|
|width|横幅の指定|
|height|縦幅の指定|
|angle|回転角の指定|
|origin|回転時の原点の指定、c: 中心、tl: 左上、tr: 右上、bl: 左下、br: 右下|
|draft|図を入れる枠だけ表示|
|clip|Bounding Box（図を入れる領域）からはみ出た部分を切り取る|
|page|複数ページあるファイル（PDFなど）の特定ページを表示する|

#### figure環境

<!-- textlint-disable ja-spacing/ja-space-between-half-and-full-width -->

- `\begin{figure} ~ \end{figure}` で挟むと図表番号を入れたりできる
  - 図を中央揃えにするとき、 `\begin{center} ~ \end{center}` で挟むのではなく、`\centering` を `\includegraphics` の前に入れる方法が良いとされている
- `\begin{figure}[htbp]` としたり、`\begin{figure}[H]` としたりすることで、図の出力位置を自分で決められる
  - 前者はh: その場(here), t: ページの上(top), b: ページの下(bottom), p: 別のページ(page)の優先順位で図を出力しようとする
    - 順番は任意で数も自由（論文だと[tbp]が多いらしい）
  - 後者はfloatパッケージが必要で、確実にその場に図を出力することができる
- `\includegraphics` の後に `\caption{図の説明}` を入れると、「図1 図の説明」のように図番号とキャプションが表示できる
- `\label{ラベル名}` をつけると、文中で `\ref{ラベル名}` としたところが番号に置き換わる

<!-- textlint-enable ja-spacing/ja-space-between-half-and-full-width -->

#### 配置

<!-- textlint-disable ja-technical-writing/ja-unnatural-alphabet -->

- 複数の図を挿入するにはminipage環境やsubcaption環境を用いると良い
- figure環境の中でそれぞれの図に関するコードを `\begin{minipage}[位置]{幅} ~ \end{minipage}` で挟む
  - 位置は図を揃える時の設定でtは上揃え、bは下揃え、なしで中央揃えとなる
  - 幅はその図が取る幅を `0.45\linewidth`（linewidthの0.45倍）のように指定する
    - 余白を考えて合計が1より少し小さくなるようにするとよい（2つ並べるのであれば0.45ずつ、3つ並べるのであれば0.3ずつなど）
  - `\linewidth` は行の長さ（カラムやミニページで変化する）、`\textwidth` は本文領域の横の長さ（カラムやミニページで不変）、`\hsize` はページの横幅を表す（この3つがよく使われていた）
- それぞれのminipage環境で `\subcaption{サブキャプション}` と書くと「(a) サブキャプション」のように表示される
  - subcaptionパッケージが必要
- またラベルをつけると `\subref{ラベル名}` としたときにサブキャプションの記号（aなど）で置き換わる
  - `\ref{ラベル名}` とすると1aのように置き換わる
- 図を複数行並べたいときは行の変わり目の `\end{minipage}` の後に `\\` をつけて、後は同じように行う

<!-- textlint-enable ja-technical-writing/ja-unnatural-alphabet -->

#### caption, subcaption

- caption, subcaptionパッケージに関して、よく紹介されているオプション（+α）を示す

```latex
\usepackage[hang, small, bf]{caption}
\usepackage[subrefformat=parens]{subcaption}
\captionsetup{compatibility=false}
```

- captionパッケージのオプションについては[ここ](https://karat5i.blogspot.com/2014/10/latex.html)や[ここ](http://abigfishinalittlepond.blogspot.com/2010/09/latexcaption.html)を見ると良い
- subcaptionパッケージのオプションについては[ここ](http://www.yamamo10.jp/~yamamoto/comp/latex/make_doc/insert_fig/index.php#SUBCAP_OP)を見ると良い
- 3行目はエラーが出たときの対処
  - [このようなエラー](http://www.yamamo10.jp/~yamamoto/comp/latex/make_doc/insert_fig/index.php#SUBCAP_ERROR)が出たときに解決できる

#### 文字の回り込み

- あまり使う機会が多くないと思ったので割愛
- 必要があれば[ここ](http://www.yamamo10.jp/~yamamoto/comp/latex/make_doc/insert_fig/index.php#WRAPFIG)を参考にすると良い

### 表

- ほぼ全てが[ここ](https://takataninote.com/tex/table.html)に書かれている

#### 基本

<!-- textlint-disable ja-technical-writing/ja-unnatural-alphabet -->

- 表は `\begin{table}[表の位置指定] ~ \end{table}` で表環境を作成する
- 表の具体的な中身は `\begin{tabular}[列の場所] ~ \end{tabular}` の間に必要なものを書いていく
- 列の場所は列ごとに指定し、列の数だけ必要になる（l: 左詰め、c: 中央揃え、r: 右揃え）
- また、`|` を入れると、その位置に縦線が引かれる
  - 例えば、`\begin{tabular}[l||cr]` とすると、3列から成り、1列目と2列目の間に2本の線が引かれた表となる
- 表の中身で、それぞれの行は `&` で列の区切り、`\\` で行の終わりを表す
- `\hline` とすると、書いた数だけその場に横線が引かれる
- `\cline{n-m}` とすると、n列目からm列目まで横線が引かれる
- 「垂直罫線を使用しない」「二重罫線を使用しない」ことで美しい表となるとされている
  - 下で説明するが、booktabsパッケージを利用するとこの規則に従って表を作成できる

<!-- textlint-enable ja-technical-writing/ja-unnatural-alphabet -->

#### セルの結合

- セルを横方向に結合したいときは `\multicolumn{結合要素数}{位置指定}{内容}` とする
  - 位置指定はtable環境のときと同じで、必要があれば `|` を入れて縦線の指定もする
- セルを縦方向に結合したいときは `\multirow{結合要素数}{#}{内容}` とする
  - multirowパッケージが必要
  - 位置指定の `#` は自動的に位置を決めるように指定するものだが、これ以外ではエラーを吐いてしまうらしい
  - 線を引くときは `\cline{}` を用いないと結合している部分にも線が引かれてしまう

#### booktabs

- booktabsを用いると美しい表が書けるとされている
  - booktabsパッケージが必要
- 主に用いるのは `\topline`（表の最上部の横線）、`\midrule`（列見出しと内容の間の横線）、`\bottomrule`（表の最下部の横線）
  - これらはそれぞれ `{幅}` をつけて幅の指定もできるが、基本的にデフォルト値でちょうどよくなるように設定されているので変える必要はない
- それ以外で用いるものは[ここ](http://www.yamamo10.jp/yamamoto/comp/latex/make_doc/table/table.php#BOOKTABS:COMMAND)に書かれている

#### 配置

- 表を横に並べるときは図と同じでminipageを用いる

### 参照

#### 基本

- 図や表、数式などで `\label{ラベル名}` したものに対して、`\ref{ラベル名}` とすると対象の図表番号を取得できる
- ラベルを付ける際は参照性を上げるために次のようにラベル名をつけると良い

|参照先の種類|ラベル名|
| -- | -- |
|図|fig:hoge|
|表|tab:hoge|
|数式|eq:hoge|
|章|chap:hoge|
|節|sec:hoge|

#### リンク

- URLを単純に書き込みたいだけならurlパッケージを使用し、`\url{URL}` とするとタイプライタ体で表示される
  - ただし、クリックしてもページが開けるようになるわけではない

#### ハイパーリンク

- ハイパーリンクをつけたいときはhyperrefパッケージを用いる（が、かなり曲者）
  - [参考1](https://www.isc.meiji.ac.jp/~mizutani/tex/link_slide/hyperlink.html)
  - [参考2](https://0-chromosome.hatenablog.jp/entry/2015/04/10/175912)
- パッケージを用いるにはプリアンブルに `\usepackage[dvipdfmx]{hyperref}` と書く
- さらにhyperrefパッケージを用いることによって生じる問題を解決するために、`\usepackage{pxjahyper}` を追加する
  - これらはパッケージリストの末尾に書いたほうが良いとされている
- こうすると参照しているところ（`\ref{}` や `\url{}` など）にハイパーリンクが埋め込まれ、参照先にジャンプできる

- 以下のようにしてハイパーリンクの関連付けが可能になる

|コマンド|内容|
| -- | -- |
|`\url{URL}`|文字列URLにURLを関連付ける|
|`\href{URL}{text}`|textをリンク文字列としてURRLと関連付ける|
|`\hypertarget{name}{text}`|textにリンクポイント名nameを付与する|
|`\hyperlink{name}{text}`|textをリンクポイント名nameと関連付ける|

- `\hyperlink{name}{text}` と書いたところから `\hypertarget{name}{text}` と書いたところに飛べる

- 上の設定のみではリンク部分全てに色付きの枠がついてしまう
  - これをなくす場合は `hidelinks` オプションをhyperrefパッケージを追加する際につければ良い
  - ただし、こうするとリンク部分とそうでない部分で見かけ上の区別が全くつかなくなる
  - 例えば、以下のようにするとURL部分のみが青くなる

```latex
\hypersetup{        % hyperrefパッケージのオプション設定
 setpagesize=false, % ページサイズが変わってしまうのを防ぐ
 colorlinks=true,   % リンクに色を付ける
 urlcolor=blue,     % 外部参照のURLは青色にする（wordに近い）
 linkcolor=black    % 内部参照は黒（＝地の文）と同じ
}
```

### 文献引用

#### 基本

- 論文などの参考文献を手で書くのは非常に面倒なので、bibtexを用いて自動化する
- まずはbibファイルに文献情報を書く

```bibtex
@article{arai2002advances,
  title={Advances in multi-robot systems},
  author={Arai, Tamio and Pagello, Enrico and Parker, Lynne E and others},
  journal={IEEE Transactions on robotics and automation},
  volume={18},
  number={5},
  pages={655--661},
  year={2002}
}
```

- author, title, publisher, yearは必須項目（厳密にはスタイルによって異なる）で、その他は自由。また、順番は適当で良い
  - Google scholarなどではbibtex形式の文献情報を表示できるのでそのままコピペするだけで済む
  - 上の例で `arai2002advances` となっている部分は本文で参照する際に用いるラベルなのでわかりやすいものをつける
  - bibファイルに書かれているが、本文では参照されていない文献があってもよい
- tex本文で参照するには `\cite{arai2002advances}` とする
  - ここに文献リストと対応した番号や著者名などが表示される
- 文献リストを表示させたい場所で次のようにする

```latex
\bibliographystyle{jplain}
\bibliography{bibファイル名（拡張子を除く）}
```

- 1行目は文献リストの表示形式を定めるもので、jplainはよく使われる形式である
  - 形式が指定されている場合はそれに従う
- 2つは順番が上の通りでなくともよく、離れていても良い
  - 実際に文献リストが表示されるのは `\bibliography{bibファイル名}` と書いたところ

#### コンパイル

- bibtexを用いている場合は普通のtexファイルのコンパイルと違い、platex -> pbibtex -> platex -> platexとコンパイルしなければならない
- vscodeを使っているならば、それ用のレシピやツールを書いておくとよい
- もしくはlatexmkを用いると自動で判断してコンパイルしてくれる

#### 文献管理ツール

- 引用する文献が増えていくとbibファイルを書くのも手間になるのでそのときは文献管理ツールを用いると良い
- bibファイルの出力だけでなく、複数デバイスでの同期やタグ付けなどもできる

### その他

#### ソースコード

- listingsやjlistingsというパッケージを用いる
  - 自分の環境では後者が入っていなかったので、[ここ](https://qiita.com/N_Matsukiyo/items/1199f07a0e1bf4fce29c)を参考にダウンロード
- ソースコードの表示は[ここ](https://yu00.hatenablog.com/entry/2015/05/14/214121)を参考にする
  - デフォルトだとほとんどきれいに表示できないので、[ここ](https://yu00.hatenablog.com/entry/2015/05/14/214121)や[ここ](https://zenn.dev/kyaon/articles/68867e2657e605)を参考にしてカスタマイズする

#### ヘッダーとフッター

- [ここ](https://texwiki.texjp.org/?LaTeX%E5%85%A5%E9%96%80%2F%E3%83%98%E3%83%83%E3%83%80%E3%83%BC%E3%81%A8%E3%83%95%E3%83%83%E3%82%BF%E3%83%BC)を参考にする
- ページ番号を記載しない場合は `\pagestyle{empty}` とする

#### VSCode関連

- [ショートカットなど](https://qiita.com/Yarakashi_Kikohshi/items/1a275f2046b002e398b3)

## 参考

- [TeX Wiki](https://texwiki.texjp.org/)
- [物理のかぎしっぽ TeX](http://hooktail.org/computer/index.php?TeX)
- [VSCodeで最高のLaTeX環境を作る](https://qiita.com/rainbartown/items/d7718f12d71e688f3573)
- [LaTeX入門](https://medemanabu.net/latex/)
  - このページを書くための参考にしたわけではないが、色々とまとまっている
