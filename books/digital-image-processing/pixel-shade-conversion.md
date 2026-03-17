---
title: "画素ごとの濃淡変換"
---

「デジタル画像処理」の第4章に該当します。

## トーンカーブ

入力画像の画素値に対する出力画像の画素値の対応をグラフで表したものをトーンカーブと呼びます。入力画像の画素値を $x$、出力画像の画素値を $y$ とすると、トーンカーブで $y = x$ より上の部分は明るく、下の部分は暗くなります。

Pythonでの実装例は以下です。トーンカーブを表す関数である `curve_fn` に入力画像を代入するだけです。

```python
def tone_curve(img, curve_fn):
    return curve_fn(img)
```

OpenCVでは、`cv2.LUT(src, lut)` でトーンカーブを用いた変換が可能です。`src` が入力画像で、`lut` が変換を表す(256,)の1次元配列です。例えば、lut[128]が入力128に対する出力を表しています。

### 折れ線型トーンカーブ

トーンカーブが直線で構成されているものです。折れ線型トーンカーブはnumpyの機能を使用すると、簡単に実装できます。

```python
def curve_fn(img):
    return np.interp(img, xs, ys)
```

`np.interp` は線形補完する関数で、`xs`、`ys` に通過したい点の座標を指定します。

以下は `xs = [0, 255*1/4, 255*3/4, 255], ys = [0, 0, 255, 255]` と指定したときの折れ線型トーンカーブ、及びこのトーンカーブで変換した画像の例です。

![折れ線型トーンカーブ](/images/digital-image-processing/chap4/line_tone_curve_graph.png =400x)
*折れ線型トーンカーブ*

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/line_tone_curve.jpg =250x) |
| ![サンプル1 白黒 ヒストグラム](/images/digital-image-processing/sample1_gray_hist.png =400x) | ![処理後 ヒストグラム](/images/digital-image-processing/chap4/line_tone_curve_hist.png =400x) |

このトーンカーブでは、暗い部分と明るい部分を0や255に貼り付け、中央部分を引き伸ばしています。変換後の画像のヒストグラムでは0や255の度数が大きく、全体が櫛状になっています。

### 累乗型トーンカーブ

折れ線型トーンカーブでは変化点の前後で性質が大きく変わり、水平な部分では濃淡変化が完全に失われてしまいます。そこで、以下の式で表される曲線のトーンカーブではその欠点を克服できます。

$$
y=255\Big(\frac{x}{255}\Big)^{\frac{1}{\gamma}}
$$

![累乗型トーンカーブ](/images/digital-image-processing/chap4/gamma_tone_curve_graph.png =400x)
*累乗型トーンカーブ*

この変換はガンマ補正とも呼ばれます。以下が $\gamma =2$ のときの変換例です。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/gamma_tone_curve.jpg =250x) |
| ![サンプル1 白黒 ヒストグラム](/images/digital-image-processing/sample1_gray_hist.png =400x) | ![処理後 ヒストグラム](/images/digital-image-processing/chap4/gamma_tone_curve_hist.png =400x) |

暗い部分が引き伸ばされてはいますが、明るい部分はある程度そのまま残っています。

### S字型トーンカーブ

S字型トーンカーブは折れ線型トーンカーブの変換例で示したトーンカーブを曲線にして滑らかにしたものです。S字型の曲線は色々ありますが、今回は、[このサイト](http://marupeke296.com/TIPS_No19_interpolation.html)で紹介されている以下の式を使用します。

$$
\begin{alignedat}{2}
    &x' = \frac{x}{255}\\
    &t = \frac{1}{2}\Bigg(1+\frac{1-e^{-a(2x'-1)}}{1+e^{-a(2x'-1)}}\cdot\frac{1+e^{-a}}{1-e^{-a}}\Bigg)\\
    &y = \big(-2t^3+3t^2\big)\cdot255
\end{alignedat}
$$

$a$ は曲がり具合を変更するパラメータです。以下にトーンカーブの例を示します。

![S字型トーンカーブ](/images/digital-image-processing/chap4/s_shape_tone_curve_graph.png =400x)
*S字型トーンカーブ*

以下は $a=4$ のときの変換例です。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/s_shape_tone_curve.jpg =250x) |
| ![サンプル1 白黒 ヒストグラム](/images/digital-image-processing/sample1_gray_hist.png =400x) | ![処理後 ヒストグラム](/images/digital-image-processing/chap4/s_shape_tone_curve_hist.png =400x) |

画素値の中央が引き伸ばされていますが、折れ線型トーンカーブのときほど0と255に集中していません。

## ヒストグラム平坦化

トーンカーブを用いた変換の目的の1つは、画像のコントラストを上げることです。そのためには、ヒストグラムの集中している部分を引き伸ばすトーンカーブを指定する必要があります。これを自動化する処理がヒストグラム平坦化です。ヒストグラム平坦化を行うと、明るさが画像ごとにある程度統一され、その後の物体検出などの処理の精度を上げることに繋がります。

総画素数を $N$、出力画像の濃淡レベルを $L$としたとき、理想的には各画素値の頻度が $N/L$ となります。しかし、入力画像のある画素値の頻度がすでに $N/L$ を超えている場合もあります。そこで、頻度が高い画素値の周辺ではまばらにして累積頻度が直線へ近づくようにします。

Pythonでの実装は以下となります。実装は[OpenCVのドキュメント](https://docs.opencv.org/4.x/d5/daf/tutorial_py_histogram_equalization.html)を参考にしています。

```python
def histogram_equalization(img):
    hist, _ = np.histogram(img[:, :, 0], bins=256)
    cdf = hist.cumsum()
    cdf_m = np.ma.masked_equal(cdf, 0)
    cdf_m = (cdf_m - cdf_m.min()) * 255 / (cdf_m.max() - cdf_m.min())
    cdf = np.ma.filled(cdf_m, 0).astype(np.uint8)

    return cdf[img]
```

以下が画像の変換例です。ヒストグラムの赤線は累積頻度となっており、見やすいよう適当にスケーリングしています。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/histogram_equalization.jpg =250x) |
| ![サンプル1 白黒 ヒストグラム](/images/digital-image-processing/sample1_gray_hist_cumsum.png =400x) | ![処理後 ヒストグラム](/images/digital-image-processing/chap4/histogram_equalization_hist.png =400x) |

暗い画像が明るめになっていることが変換後の画像から分かります。また、累積頻度が直線に近くなっています。

OpenCVでは、`cv2.equalizeHist(src)` でヒストグラム平坦化が可能です。

## 特殊な効果

### ネガ・ポジ反転

$y=-x+255$というトーンカーブを使用すると、画像の濃淡が反転します。この処理をネガ・ポジ反転と呼びます。以下はトーンカーブと変換した画像の例です。

![ネガ・ポジ反転のトーンカーブ](/images/digital-image-processing/chap4/nega_posi_reversal_graph.png =400x)
*ネガ・ポジ反転のトーンカーブ*

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/nega_posi_reversal.jpg =250x) |
| ![サンプル1 白黒 ヒストグラム](/images/digital-image-processing/sample1_gray_hist.png =400x) | ![処理後 ヒストグラム](/images/digital-image-processing/chap4/nega_posi_reversal_hist.png =400x) |

変換後の画像は濃淡が反転し、独特の風合いが出ています。ヒストグラムは変換の前後で左右対称になっています。

OpenCVでは、`cv2.bitwise_not(src)` でネガ・ポジ反転が可能です。

### ポスタリゼーション

下の図のような階段状のトーンカーブによる変換をポスタリゼーションと呼びます。この例では、画素値が4種類のみとなります。

![ポスタリゼーションのトーンカーブ](/images/digital-image-processing/chap4/postalize_graph.png =400x)
*ポスタリゼーションのトーンカーブ*

このトーンカーブを使用したポスタリゼーションの変換を下に示します。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/postalize.jpg =250x) |
| ![サンプル1 白黒 ヒストグラム](/images/digital-image-processing/sample1_gray_hist.png =400x) | ![処理後 ヒストグラム](/images/digital-image-processing/chap4/postalize_hist.png =400x) |

画像の左下付近を見比べると、グラデーションがなくなり、きっぱりと色が変わっていることがわかります。また、ヒストグラムも値が4箇所のみにしかありません。

### 2値化

ポスタリゼーションの例では4段階にわけていますが、2段階にわけたものは特に2値化と呼ばれます。以下にトーンカーブや変換例を示します。

![2値化のトーンカーブ](/images/digital-image-processing/chap4/binarize_graph.png =400x)
*2値化のトーンカーブ*

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/binarize.jpg =250x) |
| ![サンプル1 白黒 ヒストグラム](/images/digital-image-processing/sample1_gray_hist.png =400x) | ![処理後 ヒストグラム](/images/digital-image-processing/chap4/binarize_hist.png =400x) |

ヒストグラムを見ると、値が0と255になっていることがわかります。

今回は中央で白と黒を分けましたが、[大津の二値化](https://ja.wikipedia.org/wiki/%E5%A4%A7%E6%B4%A5%E3%81%AE%E4%BA%8C%E5%80%A4%E5%8C%96%E6%B3%95)のように画素値の分布に応じて自動で閾値を決定する手法もあります。

OpenCVでは、`cv2.threshold(src, threshold, maxValue, thresholdType)` で2値化が可能です。`threshold` は閾値、`maxValue` は閾値を超える画素に設定する値、`thresholdType` は2値化の方法です。`thresholdType` に `cv2.THRESH_BINARY` を設定すると単純な2値化を行います。大津の二値化を行いたい場合は、`cv2.THRESH_BINARY + cv2.THRESHOLD_OTSU` と設定します。

### ソラリゼーション

画像の濃淡を一部分反転させることでネガ画像とポジ画像が混ざったような変換をする処理をソラリゼーションと呼びます。今回は、[このサイト](https://www.momoyama-usagi.com/entry/info-img02#3-2)を参考に以下の式でトーンカーブを構成します。

$$
y = \frac{255}{2}\Bigg\{1-\cos{\Big(\frac{3\pi}{255}x\Big)}\Bigg\}
$$

トーンカーブ、変換例を示します。

![ソラリゼーションのトーンカーブ](/images/digital-image-processing/chap4/solarize_graph.png =400x)
*ソラリゼーションのトーンカーブ*

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/solarize.jpg =250x) |
| ![サンプル1 白黒 ヒストグラム](/images/digital-image-processing/sample1_gray_hist.png =400x) | ![処理後 ヒストグラム](/images/digital-image-processing/chap4/solarize_hist.png =400x) |

## カラー画像の変換

RGB画像に対してもトーンカーブによる変換が可能です。しかし、RGBに同じトーンカーブを適用した場合、想定した色とならない場合もあります。極端な例として、以下に[ソラリゼーション](#ソラリゼーション)のトーンカーブで変換した例を示します。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_color.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/solarize_color.jpg =250x) |

特定のチャンネルにのみトーンカーブを適用した場合は、その色を強調できます。

疑似カラー画像もトーンカーブの利用で作成できます。グレースケール画像に対して、RGBで異なるトーンカーブを適用するだけです。以下にトーンカーブの例と変換結果を示します。

![疑似カラーのトーンカーブ](/images/digital-image-processing/chap4/pseudo_color_graph.png =400x)
*疑似カラーのトーンカーブ*

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/pseudo_color.jpg =250x) |

トーンカーブは折れ線型トーンカーブを応用して作成しています。

## 複数画像の利用

複数の入力画像の同じ位置の画素ごとに演算処理を行い、出力画像の画素値を計算する処理を画像間演算といいます。演算は算術演算や論理演算などが使われます。

### アルファブレンディング

次の式のように、2つの画素の重み付き平均で出力画像の画素値を計算する方法をアルファブレンディングといいます。

$$
o = \alpha i_1 + (1-\alpha) i_2
$$

以下に$\alpha=0.5$での例を示します。

| 元画像1 | 元画像2 | 変換後 |
| ---- | ---- | ---- |
| ![サンプル1 カラー](/images/digital-image-processing/sample1_color.jpg =250x) | ![サンプル2 カラー](/images/digital-image-processing/sample2_color.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/alpha_blend.jpg =250x) |

動画で$\alpha$の値を時間的に変化させることで別のシーンに段々移行するような見せ方もでき、ディゾルブなどと呼ばれます。

OpenCVでは、`cv2.addWeighted(src1, alpha, src2, beta, gamma)` で実行可能です。出力画像の画素値は $\text{src1}*\alpha+\text{src2}*\beta+\gamma$ となります。

### エンボス

エッジを強調する処理をエンボスと呼びます。エンボス画像は、入力画像（$i_1$）とネガ・ポジ反転した上で数画素平行移動した画像（$i_2$）に以下の画像間演算で生成します。

$$
o = i_1 + i_2 - 128
$$

ただし、出力画像の画素値は[0 255]でクリッピングします。

Pythonでの実装は以下です。

```python
def emboss(img, dx = 2, dy = 2):
    # ネガポジ反転
    reversaled = nega_posi_reversal(img)

    # 反転後の画像を移動
    moved = np.zeros_like(reversaled)
    if dx >= 0 and dy >= 0:
        moved[dx:, dy:] = reversaled[: reversaled.shape[0] - dx, : reversaled.shape[1] - dy]
    elif dx >= 0 and dy < 0:
        moved[dx:, : moved.shape[1] + dy] = reversaled[: reversaled.shape[0] - dx, -dy:]
    elif dy < 0 and dy >= 0:
        moved[: moved.shape[0] + dx, dy:] = reversaled[-dx:, : reversaled.shape[1] - dy]
    else:
        moved[: moved.shape[0] + dx, : moved.shape[1] + dy] = reversaled[-dx:, -dy:]

    return np.clip(img + moved - 128, 0, 255)
```

生成例はこちらです。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![処理後](/images/digital-image-processing/chap4/emboss.jpg =250x) |

建物の輪郭などのエッジが強調されています。

## 参考

* [ゲームつくろー！ < Programming TIPs編 < その19 補間関数あれこれ](http://marupeke296.com/TIPS_No19_interpolation.html)
* [OpenCVドキュメント Histograms - 2: Histogram Equalization](https://docs.opencv.org/4.x/d5/daf/tutorial_py_histogram_equalization.html)
* [工業大学生ももやまのうさぎ塾 うさぎでもわかる画像処理　Part02　トーンカーブと画像処理 [Python・MATLABコード付き]](https://www.momoyama-usagi.com/entry/info-img02#3-2)
