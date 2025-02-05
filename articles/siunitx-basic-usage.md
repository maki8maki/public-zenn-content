---
title: "siunitxの使い方"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["latex", "siunitx"]
published: true
---

## はじめに

siunitxがバージョン3になってから `\SI` や `\si` コマンドが非推奨になりました（使用自体は現時点で可能です）。しかし、この変更に対応した記事が少なく、よく参考にしていた記事も見られなくなってしまったため、ここに改めて整理します。

以下のレポジトリで簡単な例を紹介しています。

https://github.com/maki8maki/latex-example

## siunitxとは

「数値と単位との間にスペースを空けること」、「単位は必ず立体にすること」など国際単位系として決められたルールが多くあり、これらに従って文書を書くのは非常に面倒です。siunitxは簡単に行えるパッケージです。

## 基本的な使い方

最低限知っておくべきコマンドは以下の3つです。オプションは任意で設定する項目です。

* `\qty[オプション]{数値}{単位}`：単位と数字を出力する
* `\unit[オプション]{単位}`：単位のみを出力する
* `\num[オプション]{数値}`：数字のみを出力する

`\qty` は文中、`\unit` は表の見出しや数式中でよく使われます。`\num` は一見使い所が分かりづらいですが、数値を整えて出力してくれるので文中で使う機会が意外とあります。また、`\qty` と `\unit`、`\unit` と `\num` ではそれぞれ単位と数値が入力・出力ともに同じ扱いです。

単位は様々な書き方ができます。例えば、キログラムの場合は `kg`、`\kilogram`、`\kilo\gram` いずれの書き方でも問題有りません。

数値に入力した値は適切に整えられて出力されます。デフォルトでは、大きい数値の場合には$30\ 000$のように3桁区切りでスペースが入ります。また、3e4と入力すると、$3\times10^4$と出力してくれます。

## 単位

### SI基本単位

|コマンド|単位|出力|
| -- | -- | -- |
|\ampere|アンペア|A|
|\candela|カンデラ|cd|
|\kelvin|ケルビン|K|
|\kilogram|キログラム|kg|
|\metre（\meter）|メートル|m|
|\mole|モル|mol|
|\second|秒|s|

### その他のSI単位

以下は一例で、他にも多くの単位が用意されています。また、eV（エレクトロンボルト）など非SI単位系で使用できるものもあります。

|コマンド|単位|出力|
| -- | -- | -- |
|\becquerel|ベクレル|Bq|
|\degreeCelsius|セルシウス度|℃|
|\hertz|ヘルツ|Hz|
|\joule|ジュール|J|
|\newton|ニュートン|N|
|\ohm|オーム|Ω|
|\pascal|パスカル|Pa|
|\radian|ラジアン|rad|
|\volt|ボルト|V|
|\watt|ワット|W|

### 接頭辞

下記は一部の抜粋で、さらに大きい・小さい接頭辞も用意されています。

|コマンド|単位|出力|
| -- | -- | -- |
|\tera|$10^{12}$ テラ|T|
|\giga|$10^9$ ギガ|G|
|\mega|$10^6$ メガ|M|
|\kilo|$10^3$ キロ|k|
|\hecto|$10^2$ ヘクト|h|
|\deca|$10^1$ デカ|da|
|\deci|$10^{−1}$ デシ|d|
|\centi|$10^{−2}$ センチ|c|
|\milli|$10^{−3}$ ミリ|m|
|\micro|$10^{−6}$ マイクロ|μ|
|\nano|$10^{−9}$ ナノ|n|
|\pico|$10^{−12}$ ピコ|p|

### 組立単位

ひとつの組立単位でも複数の書き表し方が可能です。以下はニュートンの例です。1つ目と2つ目は$\mathrm{kg}\ \mathrm{m}\ \mathrm{s}^{-1}$と表示されます。そして、3つ目は$\mathrm{kg}\ \mathrm{m}/\mathrm{s}$、4つ目が$\frac{\mathrm{kg}\ \mathrm{m}}{\mathrm{s}}$と表示されます。

```latex
\unit{kg.m.s^{-1}}
\unit{\kilogram\meter\per\second}
\unit[per-mode=symbol]{\kilogram\meter\per\second}
\unit[per-mode=fraction]{\kilogram\meter\per\second}
```

### その他の単位

`\ang{数値}` で角度、`\qty{数値}{\percent}` でパーセントの表示が可能です。

## 数値

上でも述べたようにデフォルトでは入力したものを整形するのみでそのまま出力されます。オプションの指定によってより便利な書き方もできます。以下に例を挙げます。

`exponent-mode=scientific` を指定すると、自動で指数表記されます。下の例では、$1.00\times10^3$となります。

`group-separator={,}` を指定すると桁区切りを変更できます。下の例では、10,000となります。少数以下も同様の区切りが適用されることに注意してください。

```latex
\num[exponent-mode=scientific]{1000}
\num[group-separator={,}]{10000}
```

## その他のコマンド

### 複素数

`\complexqty[オプション]{数値}{単位}`、`\complexnum[オプション]{数値}` で複素数の表示が可能です。

### リスト

`\qtylist`、`\numlist` で簡単なリストの表示が可能です（オプションの指定などは `\qty`、`\num` と同じ）。ただし、デフォルトのままだと、a, b, and cのように英語で表記されます。list-final-separatorとlist-separatorというオプションを指定することで区切りを変更できます。前者はリストの最後の区切り、後者はそれ以外の区切りを指定できます。

```latex
\qtylist[list-final-separator = {, }]{0.13;0.67;0.80}{mm} % -> 0.13 mm, 0.67mm, 0.80 mm
```

### 範囲

`\qtyrange`、`\numrange` で簡単なリストの表示が可能です（オプションの指定などは `\qty`、`\num` と同じ）。ただし、デフォルトのままだと、a to bのように英語で表記されます。range-phrageというオプションの指定で間の表現を変更可能です。

```latex
\qtyrange[range-phrase={--}]{5}{100}{mm} % -> 5 mm–100 mm
```

### 掛け算・べき乗

`\qtyproduct`、`\numprduct` で掛け算を簡単にできます。通常、掛け算の記号の表記には `times` が必要ですが、`x（エックス）` のみでできるようになります。

```latex
\qtyproduct{2x3x4}{cm} % -> 2 cm × 3 cm × 4cm
```

べき乗は単に `^{}` とするだけでなく、`square`（2乗）、`cubic`（3乗）、`tothe{}`（任意の値）などのマクロが用意されています。

### オプション指定

コマンドを書くたびにオプションをいちいち書くのは非常に面倒です。オプションを\usepackageで指定すると文書全体に適用され、\sisetupコマンドで指定すると以降の文章に適用されます。一方で、コマンドで指定されたものはそのコマンドのみに有効です。

```latex
\documentclass[11pt]{jsarticle}

\usepackage[オプション1]{siunitx} % オプション1は文書全体に適用される

\begin{document}

\qty[オプション2]{32}{cm} % オプション1と2が適用、2はこのコマンドのみ

\sisetup{オプション3} % オプション3は以降適用される

\num{4e5} % オプション1と3が適用される

\end{document}
```

## 参考

* [siunitx ユーザーマニュアル](https://mirror.aria-on-the-planet.es/CTAN/macros/latex/contrib/siunitx/siunitx.pdf)
* [SI単位（国際単位系） - siunitxパッケージの使い方](https://medemanabu.net/latex/package/)
* [SI単位（国際単位系） - siunitxパッケージのマクロ](https://medemanabu.net/latex/macro/)
* [LaTeXの使い方 物理量したい（siunitx）](https://kumaroot.readthedocs.io/ja/latest/latex/latex-siunitx.html)
* [実験AのためのLaTeX小技集 単位](https://uec.medit.link/latex/units.html)
