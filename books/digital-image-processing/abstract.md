---
title: "はじめに"
---

## 背景

学生のときに講義でさらっと流した画像処理について改めて学び直そうと思い、「[デジタル画像処理](https://www.cgarts.or.jp/books_detail/eip_2/)」で紹介されている処理手法を自分で実装しました。その内容を本にまとめています。手元にあるのが本は改定第二版であり、参照先もこの版にしたがっています。
実装した場合のコードと一緒に、同様の機能を持つ、OpenCVやPillowなどのライブラリの関数も紹介します。

:::message
実装したコードと紹介OpenCVやPillowの関数の引数や結果が厳密に一致することは保証しないので注意してください。
:::

実装したコードは以下に上げています。

https://github.com/maki8maki/digital-image-processing

## 環境

* Python3.14.0

手法の実装はNumpyを使用します。

## サンプル画像

以下のサンプル画像を使用します。白黒画像を使用する場合は、画像編集ソフトで作成しています。

| サンプル1 | サンプル2 |
| ---- | ---- |
| ![サンプル1 カラー](/images/digital-image-processing/sample1_color.jpg =250x) | ![サンプル2 カラー](/images/digital-image-processing/sample2_color.jpg =250x) |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | |
