---
title: "空間に基づく濃淡変換（空間フィルタリング）"
---

「デジタル画像処理」の第5章に該当します。

## 空間フィルタリング

前章では、画素ごとに濃淡を変換する手法を紹介しました。空間フィルタリングは、注目画素の主に周囲の画素を使用して出力する画素値を決定します。

空間フィルタリングの一種である線形フィルタは、以下の式で計算されます。

$$
g(i, j) = \sum_{n=-W}^{W}\sum_{m=-W}^{W}f(i+m, j+n)\ h(m, n)
$$

ここで、$f(i, j)$が入力画像、$g(i, j)$が出力画像、$h(m, n)$がフィルタを表します。フィルタのサイズは$(2W+1)\times(2W+1)$です。

上記の式に当てはまらないものは非線形フィルタと呼ばれます。

線形フィルタのPythonでの実装例は以下です。

```python
def linear_filtering(img, filter):

    hh, wh = h // 2, w // 2
    padded_img = np.pad(img, ((hh, hh), (wh, wh), (0, 0)), mode="constant")
    filter = filter[..., np.newaxis] # 軸を拡張
    output_img = np.zeros(img.shape, dtype=np.float64) # 計算結果が落ちないようにfloatにする

    for i in range(img.shape[0]):
        for j in range(img.shape[1]):
            output_img[i, j] = np.sum(padded_img[i : i + h, j : j + w] * filter, axis=(0, 1))

    # 入力画像と同じ範囲、型とする
    return np.clip(output_img, np.iinfo(img.dtype).min, np.iinfo(img.dtype).max).astype(img.dtype)
```

線形フィルタで出力画像を計算する際に、画像の端ではフィルタの範囲に画素がなくなるためそのままだとエラーとなってしまいます。対処方法として、計算できない領域は飛ばす（その分出力画像が小さくなる）、入力画像を拡張して何らかの値で埋めるなどがあります。ここでは、入力画像を画素値ゼロで拡張する方法を取ります。`np.pad` はこの処理を行う関数です。

上記の実装では画素ごとにループを回していますが、Pythonでのこの実装方法は計算速度が遅いです。OpenCVでは、`cv2.filter2D(src, ddepth, kernel)` でフィルタを使用した変換が高速にできます。`src` が入力画像で、`ddpeth` が出力画像の型、`kernel` がフィルタを表します。

## 平滑化

滑らかな画像を得るための処理を平滑化と呼びます。画像に含まれるノイズなどを軽減する目的で用いられます。

### 平均化

フィルタによって覆われる領域の平均値を求める処理が平均化です。$m\times n$のサイズのフィルタの値は、$h(m, n) = \frac{1}{m n}$となります。通常は$m = n$で使用され、フィルタサイズが大きいほど平滑化の効果が強くなります。

フィルターのPythonでの実装例は以下となります。

```python
filter = np.full(size, 1.0 / size[0] / size[1]) # 例：size = (5, 5)
```

また、以下が `size = (21, 21)` としたときの変換例です。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 カラー](/images/digital-image-processing/sample1_color.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/averaging_filtering.jpg =250x) |

OpenCVでは、`cv2.blur(src, ksize)` で平均化フィルタをかけられます。`ksize` はフィルタサイズです。

### 重み付き平均化

フィルタの中央に近いほど大きな重みにして平均値を取る加重平均化フィルタもあります。その中でも重みをガウス分布に近づけたものをガウシアンフィルタと呼びます。平均0、標準偏差σとしたときの2次元ガウス分布は以下で表せます。

$$
h(x, y) = \frac{1}{2\pi\sigma} \exp{\Big(-\frac{x^2+y^2}{2\sigma^2}\Big)}
$$

単純な平均化フィルタと見た目に大きな違いは出ないですが、より滑らかな平滑化が期待できます。

Pythonでの実装例は以下です。

```python
filter = np.zeros(size)

hc, wc = size[0] // 2, size[1] // 2

for i in range(size[0]):
    for j in range(size[1]):
        dh = i - hc
        dw = j - wc
        filter[i][j] = math.exp(-(dh**2 + dw**2) / (2 * sigma**2)) / (2 * math.pi * sigma**2)
filter /= np.sum(filter) # 合計が1になるように正規化
```

以下が `size = (21, 21), sigma = 3.0` としたときの変換例です。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 カラー](/images/digital-image-processing/sample1_color.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/gaussian_filtering.jpg =250x) |

OpenCVでは、`cv2.GaussianBlur(src, ksize, sigmaX)` でガウシアンフィルタをかけられます。

### 特定方向の平滑化

平滑化を特定方向に行うことも可能です。斜め方向の場合はフィルターの対角線上、上下左右方向の場合はフィルターの中央行・列のみに値を設定することで実現できます。

Pythonでの実装例は以下です。

```python
# 方向に応じたフィルタを作成する
filter_value = 1.0 / size
if direction == "vertical":
    filter = np.full((size, 1), filter_value)
elif direction == "horizontal":
    filter = np.full((1, size), filter_value)
elif direction == "right diagonal":
    filter = np.zeros((size, size))
    for i in range(size):
        filter[size - i - 1, i] = filter_value
elif direction == "left diagonal":
    filter = np.zeros((size, size))
    for i in range(size):
        filter[i, i] = filter_value
```

以下がサイズ21、左斜め方向のときの変換例です。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 カラー](/images/digital-image-processing/sample1_color.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/directional_smoothing.jpg =250x) |

少し分かりづらいですが、画像の左下にある円形の模様に注目すると、斜め方向に引き伸ばされていることがわかります。

## エッジ抽出

画像内の明るさが急激に変わるエッジ部分を取り出す処理をエッジ抽出と呼びます。

### 微分フィルタ

微分フィルタは隣接画素の差分を取ることでエッジを抽出します。縦方向と横方向で別のフィルタとなります。

```text:横方向
      [ 0 0 0]
0.5 x [-1 0 1]
      [ 0 0 0]
```

```text:縦方向
      [0  1 0]
0.5 x [0  0 0]
      [0 -1 0]
```

フィルタ内にマイナスの値が含まれるので、出力画像の画素値もマイナスの値を取り得ます。フィルタの適用結果をわかりやすく表示するために、値がプラスの場合とマイナスの場合で別の色に着色する方法が使われます。今回はプラスの値を黄色、マイナスの値をシアンで表示します。

横方向の差分を$\Delta_x f(i, j)$、縦方向の差分を$\Delta_y f(i, j)$としたときに、$\Big(\Delta_x f(i, j), \Delta_y f(i, j)\Big)$が画素値の勾配となります。さらに、勾配の大きさ$m(i, j)$と方向$\theta(i, j)$は次のようになります。

$$
m(i, j) = \sqrt{\Big(\Delta_x f(i, j)\Big)^2 + \Big(\Delta_y f(i, j)\Big)^2} \\
\theta(i, j) = \tan^{-1}{\frac{\Delta_y f(i, j)}{\Delta_x f(i, j)}}
$$

Pythonでの実装例は以下です。

```python
# 正負での色付け
def colorize_pixel_sign(img):
    pos = np.where(img > 0)
    neg = np.where(img < 0)
    out = np.zeros((*img.shape[:2], 3), dtype=np.uint8)

    # 元画像の画素値の絶対値に応じて色の強さを変える
    out[*pos[:2], :2] = img[*pos][..., np.newaxis]
    out[*neg[:2], 1:] = np.abs(img[*neg][..., np.newaxis])

    # 最大値がuint8の最大値と一致するようにスケーリング
    out = out.astype(np.float64) / int(out.max()) * np.iinfo(np.uint8).max

    return out

# 勾配の大きさを求める
def calc_norm(img, vertical_filter, horizontal_filter):
    tmp_img = img.astype(np.int16)  # マイナス値も取れるように型を変える

    # 縦方向と横方向のフィルタをかける
    vertical_out = linear_filtering(tmp_img, vertical_filter).astype(np.float64)
    horizontal_out = linear_filtering(tmp_img, horizontal_filter).astype(np.float64)

    # 勾配の大きさを計算し、スケーリングする
    out = np.sqrt(vertical_out**2 + horizontal_out**2).astype(np.float64)

    # 最大値がuint8の最大値と一致するようにスケーリング
    out = out / int(out.max()) * np.iinfo(np.uint8).max
    return out.astype(np.uint8) * np.ones(3, dtype=np.uint8)  # RGB形式に変換する
```

以下が変換例です。

| 元画像 | 横方向 | 縦方向 | 勾配の大きさ |
| ---- | ---- | ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![横方向](/images/digital-image-processing/chap5/derivative_filtering_horizontal.jpg =250x) | ![縦方向](/images/digital-image-processing/chap5/derivative_filtering_vertical.jpg =250x) | ![勾配の大きさ](/images/digital-image-processing/chap5/derivative_filtering_norm.jpg =250x) |

左下の縦に隙間が空いている箇所を確認すると、横方向と縦方向でエッジの抽出結果が異なります。また、勾配の大きさはこれらを合わせたものであることもわかります。

### プリューウィットフィルタ、ソーベルフィルタ

上で紹介した微分フィルタではノイズも強調されてしまいます。そこで、微分したあとに、微分の方向と直行する方向に平滑化してノイズを低減する手法が提案されています。微分と平滑化の2つのフィルタは以下のように1つのフィルタで表せ、これがプリューウィットフィルタと呼ばれます（正確には1/6を除いた整数で表されるフィルタ）。

```text:横方向
      [ 0 0 0]         [0 1 0]         [-1 0 1]
0.5 x [-1 0 1] & 1/3 x [0 1 0] → 1/6 x [-1 0 1]
      [ 0 0 0]         [0 1 0]         [-1 0 1]
```

また、平滑化を行う際に、中央に重みを付けた平滑化を行う方法もあり、こちらはソーベルフィルタと呼ばれます（正確には1/8除いた整数で表されるフィルタ）。

```text:横方向
      [ 0 0 0]         [0 1 0]         [-1 0 1]
0.5 x [-1 0 1] & 1/4 x [0 2 0] → 1/8 x [-2 0 2]
      [ 0 0 0]         [0 1 0]         [-1 0 1]
```

また、以下がそれぞれのフィルタで勾配の大きさを求めた結果です。

| 元画像 | プリューウィットフィルタ | ソーベルフィルタ |
| ---- | ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![プリューウィットフィルタ](/images/digital-image-processing/chap5/prewitt_filtering.jpg =250x) | ![ソーベルフィルタ](/images/digital-image-processing/chap5/sobel_filtering.jpg =250x) |

今回のサンプル画像では微分フィルタとプリューウィットフィルタ・ソーベルフィルタで大きな違いは見られませんでした。もう少し粗い画像やわざとノイズをかけると効果が見えるかもしれません。

OepnCVでは、恐らくプリューウィットフィルタはそのままの関数はありませんが、`cv2.Sobel(src, ddpeth, dx, dy)` でソーベルフィルタをかけられます。`dx` と `dy` は水平・垂直方向の微分次数を表し、`dx = 1, dy = 0` とすると、水平方向の勾配、逆にすると垂直方向の勾配を求められます。

### 2次微分とラプラシアン

微分フィルタやプリューウィットフィルタ、ソーベルフィルタは1次微分に相当します。2次微分を求めるフィルタは次のようになります。

```text:横方向
[0  0 0]
[1 -2 1]
[0  0 0]
```

横方向と縦方向の2次微分を足し合わせるとラプラシアンの値が得られます。フィルタで表すと次になります。

```text:
[0  1 0]
[1 -4 1]
[0  1 0]
```

以下が変換例です。こちらも値がプラスの箇所とマイナスの箇所で色を変えています。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/laplacian_filtering.jpg =250x) |

ラプラシアンでは、方向に依存しないエッジが直接得られます。しかし、微分を繰り返すことになるので、今回の例では分かりづらいですが、ノイズを強調してしまいます。そのため、ガウシアンフィルタで平滑化を行った後にラプラシアンフィルタでエッジを得るということが行われます。この処理は次の式で表すことができます。

$$
h(x, y) = \frac{x^2 + y^2 - 2\sigma^2}{2\pi\sigma^2}\exp{\Big(-\frac{x^2 + y^2}{2\sigma^2}\Big)}
$$

これをフィルタ化したものがLoGフィルタと呼ばれます。

以下がPythonでの実装例です。

```python
def int16_to_uint8(img):
    uint8_max_half = int(np.iinfo(np.uint8).max / 2)

    # スケーリングする基準の値
    compared_value = int(abs(img).max())

    #int16における0がuint8の中央になるようにスケーリング
    out = img * uint8_max_half / compared_value + uint8_max_half
    return out.astype(np.uint8)

def LoG_filtering(img, size, sigma):
    filter = np.zeros(size)

    hc, wc = size[0] // 2, size[1] // 2

    for i in range(size[0]):
        for j in range(size[1]):
            dh = i - hc
            dw = j - wc
            filter[i][j] = (
                math.exp(-(dh**2 + dw**2) / (2 * sigma**2)) * (dh**2 + dw**2 - 2 * sigma**2) / (2 * math.pi * sigma**6)
            )

    filter /= np.sum(np.abs(filter))  # 正規化

    tmp_img = img.astype(np.int16)  # 正負があるため型を変換
    filtered = linear_filtering(tmp_img, filter)
    return int16_to_uint8(filtered)
```

`size = (21, 21), sigma = 3.0` での変換例が以下です。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/Log_filtering.jpg =250x) |

OpenCVでは、LoGフィルタを直接行う関数はなさそうなので、`cv2.GaussianBlur(...)` と `cv2.Laplacian(src, ddpeth)` を順に実行する必要があります。

LoGフィルタは異なるσのガウシアンフィルタの差で近似できることが知られており、これをDoGフィルタと呼ばれます。Pythonでの実装例を下に示します。

```python
def DoG_filtering(img, size, sigma, k):
    filter = np.zeros(size)

    hc = int(size[0] / 2)
    wc = int(size[1] / 2)

    s2 = k * sigma

    for i in range(size[0]):
        for j in range(size[1]):
            dh = i - hc
            dw = j - wc
            filter[i][j] = math.exp(-(dh**2 + dw**2) / (2 * s2**2)) / (2 * math.pi * s2**2) - math.exp(-(dh**2 + dw**2) / (2 * sigma**2)) / (2 * math.pi * sigma**2)

    filter /= np.sum(np.abs(filter))  # 正規化

    tmp_img = img.astype(np.int16)  # 正負があるため型を変換
    filtered = linear_filtering(tmp_img, filter)
    return int16_to_uint8(filtered)
```

`size = (21, 21), sigma = 3.0, k=1.6` での変換例が以下です。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/Dog_filtering.jpg =250x) |

DoGフィルタもOpenCVではそのものの関数はなさそうなので、σを変えた2つの `cv2.GaussianBlur(...)` の結果の差を求めることで実装できます。

### エッジ検出

エッジ抽出処理の出力は実数の値を持つ画像であるため、どの画素がエッジであるかを特定する必要があります。

#### LoGフィルタのゼロ交差によるエッジ検出

下は画像の1列に着目したときの1次微分・2次微分の結果です。2次微分では、エッジの両側に値がプラスとマイナスのペアで現れます。この変化の間に位置する値が0になる位置（ゼロ交差）の画素をエッジみなせます。

```text
元画像 ：[..., 0,  0,  5,  10, 10, ...]
1次微分：[..., 0,  5, 10,   5,  0, ...]
2次微分：[..., 5, 10,  0, -10, -5, ...]
```

ただし、値が0となる位置をエッジとしてみなすと、明るさが一定の領域でも多くの誤検出が発生します。そこで、画素値の勾配の大きさが一定値以上のもののみを残す閾値処理を行うと、誤検出を減らすことができます。

Pythonでの実装例を下に示します。

```python
def LoG_filter_edge_extraction(img, size, sigma, threshold):
    # LoG_filteringの結果はuint8の中央と元の結果の0が
    # 一致するようにスケーリングされているので、その分の値を引く
    filtered = (LoG_filtering(img, size, sigma).astype(np.float64) - np.iinfo(np.uint8).max / 2)[:, :, 0]

    pad_filtered = np.pad(filtered, 1, "edge")

    out = np.ones_like(img) * np.iinfo(np.uint8).max
    for i in range(img.shape[0]):
        for j in range(img.shape[1]):
            # 3x3を切り出す
            arr = pad_filtered[i : i + 3, j : j + 3]
            if arr[1, 0] * arr[1, 2] < 0 or arr[0, 1] * arr[2, 1] < 0:
                # 上下または左右の画素で正負が異なる場合、エッジの候補となる
                if not threshold or abs(arr[1, 1]) > threshold:
                    # エッジとなる画素はゼロを設定する
                    out[i, j] = 0

    return out
```

`size = (21, 21), sigma = 3.0, threshold = 1` での変換例が以下です。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/LoG_filter_edge_extraction.jpg =250x) |

ゼロ交差の検出を直接行うOpenCVの関数は特に見当たりませんでした。

#### ケニーのエッジ検出

上で紹介したような単純な閾値処理によるエッジ検出では、エッジが不連続になったり、誤検出されたりします。ケニーのエッジ検出は以下の方法を取ることで、これらをなるべく含まないようにしています。

**1. ノイズ低減と微分**
最初にガウシアンフィルタを適用してノイズを低減します。標準偏差を大きくするほどノイズの低減効果は高まりますが、エッジが不正確になります。次に横方向と縦方向の微分値を画素ごとに求め、勾配の大きさと方向を以下の式で求めます。

$$
m(i, j) = \sqrt{\Big(f_x(i, j)\Big)^2 + \Big(f_y(i, j)\Big)^2} \\
\theta(i, j) = \tan^{-1}{\frac{f_x(i, j)}{f_y(i, j)}}
$$

**2. 勾配の大きさが最大となる位置の検出**
画像の濃淡に幅がある場合、勾配の大きさのみから抽出するとエッジも幅を持つ可能性があります。そこで、勾配の方向に沿って勾配の大きさを調べ、位置$(i, j)$で最大となるときにエッジの候補とします。この処理はゼロ交差を求める処理に該当します。

勾配の方向に沿った最大値であるかを判定する処理では、勾配の方向に伸ばした先の画素値を周囲の画素から求めた補間値を使用するのがいいとされています。しかし、ここでは簡単のために最も近い画素の画素値（ニアレストネイバー）を使用する実装を紹介しています。

**3. 閾値処理**
上で検出したエッジ候補の中からエッジを選びます。1つの閾値を設け、勾配の大きさがそれ以上のものを選ぶ場合ではエッジが途切れ途切れになります。ケニーのエッジ検出では2つの閾値を使用します。上側閾値$t_h$よりも大きい場合はエッジとみなし、下側閾値$t_l$よりも小さい場合はエッジでないとみなします。$t_h$と$t_l$の間の場合はエッジとされた画素に隣接している場合のみエッジと判断します。

以下にPythonでの実装例を示します。

```python
def canny_edge_detection(img, sigma, th_low, th_high):
    # 1次元のみを取り出す
    tmp_img = (img[:, :, 0])[..., np.newaxis]

    # ノイズ低減
    smoothed = gaussian_filtering(tmp_img, sigma=sigma).astype(np.int16)

    vertical_filter, horizontal_filter = get_sobel_filter()
    fx = linear_filtering(smoothed, vertical_filter).astype(np.float64)
    fy = linear_filtering(smoothed, horizontal_filter).astype(np.float64)

    m = np.sqrt(fx**2 + fy**2).astype(np.float64)  # 勾配の大きさ
    theta = np.atan2(fy, fx).astype(np.float64)  # 勾配の方向

    # −90度から90度に変換
    theta[np.where(theta > math.pi / 2)] -= math.pi
    theta[np.where(theta < math.pi / 2)] += math.pi

    th1 = math.atan(0.5)  # 0度と±45度をわける閾値
    th2 = math.atan(2.0)  # ±45度と±90度をわける閾値

    candidates = [] # エッジ候補
    for i in range(1, smoothed.shape[0] - 1):
        for j in range(1, smoothed.shape[1] - 1):
            if np.abs(theta[i, j]) <= th1:
                # 方向が0度に近い場合
                ma = m[i + 1, j]
                mb = m[i - 1, j]
            elif (th1 < theta[i, j]) & (theta[i, j] <= th2):
                # 角度が45度に近い場合
                ma = m[i + 1, j + 1]
                mb = m[i - 1, j - 1]
            elif th2 < np.abs(theta[i, j]):
                # 角度が±90度に近い場合
                ma = m[i, j + 1]
                mb = m[i, j - 1]
            else:
                # 角度が-45度に近い場合
                ma = m[i + 1, j - 1]
                mb = m[i - 1, j + 1]

            if m[i, j] >= ma and m[i, j] >= mb:
                # 勾配方向で該当画素の勾配の大きさが最大の場合にエッジの候補とする
                candidates.append([i, j])

    out = np.ones_like(img) * np.iinfo(np.uint8).max
    for i, j in candidates:
        # 閾値処理
        if m[i, j] >= th_high:
            # th_highより大きい場合はエッジとみなす
            out[i, j] = 0
        elif m[i, j] >= th_low:
            # th_lowより大きい場合は隣接画素のいずれかがth_highより大きい場合のみエッジとみなす
            if np.max([m[i + 1, j], m[i - 1, j], m[i, j + 1], m[i, j - 1]]) >= th_high:
                out[i, j] = 0

    return out
```

`sigma = 0.3, th_low = 10, th_high = 30` での変換例が以下です。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/canny_edge_detection.jpg =250x) |

OpenCVでは、`cv2.Canny(src, threshold1, threshold2)` でケニーのエッジ検出を行えます。

## 鮮鋭化

エッジ抽出は画像のエッジのみを取り出す処理ですが、元の画像の濃淡を残したままエッジを強調する処理を鮮鋭化と呼びます。アンシャープマスキングでは次のように鮮鋭化を行います。

まず、元画像（$i_{in}$）に平滑化処理を施した画像（$i_{smoothed}$）を元画像から引きます。この結果を定数倍（$k$）した上で元画像と足し合わせることで出力画像（$i_{out}$）を得ます。式にするとこうなります。

$$
i_{out} = i_{in} + k \Big(i_{in} - i_{smoothed}\Big)
$$

一連の処理を1つのフィルタで表すこともできます。サイズが3x3の場合は次のようになります。

```text:
[-k/9   -k/9 -k/9]
[-k/9 1+8k/9 -k/9]
[-k/9   -k/9 -k/9]
```

以下がPythonでの実装例です。

```python
def unsharp_masking(img, k):
    normal_filter = np.zeros((3, 3))
    normal_filter[1, 1] = 1

    smoothing_filter = np.full((3, 3), 1.0 / 9.0)

    # フィルタを計算で求める
    filter = normal_filter + k * (normal_filter - smoothing_filter)

    return linear_filtering(img, filter)
```

`k = 4` での変換例がこちらです。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒](/images/digital-image-processing/sample1_gray.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/unsharp_masking.jpg =250x) |

OpenCVでは関数は用意されていないようなので、フィルタを作成して自分で適用する必要があります。

## エッジを保存した平滑化

平滑化を行うことでノイズの低減が可能ですが、エッジも滑らかにしてしまいます。以下ではエッジを保ちつつノイズを低減する手法を説明します。

### バイラテラルフィルタ

バイラテラルフィルタは、ガウシアンフィルタで行われた注目画素からの距離による重みに加え、注目画素との画素値の差による重みを付けて平均化するフィルタです。式にすると以下になります。

$$
g(i, j) = \frac{\sum_{n=-W}^{W}\sum_{m=-W}^{W}w(i, j, m, n)f(i+m, j+n)}{\sum_{n=-W}^{W}\sum_{m=-W}^{W}w(i, j, m, n)} \\
w(i, j, m, n) = \exp{\Big(-\frac{m^2+n^2}{2\sigma_1^2}\Big)}\exp{\Big(-\frac{\big(f(i, j)-f(i+m, j+n))^2}{2\sigma_2^2})}
$$

$w(i, j, m, n)$の1つ目の項が距離による重みを表し、2つ目の項が画素値の差による重みを表しています。

Pythonでの実装例は次のようになります。

```python
def bilateral_filtering(img, size, sigma1, sigma2):
    hc, wc = size[0] // 2, size[1] // 2

    # 位置についてのガウシアンフィルタを作成
    gaus_filter = np.zeros(size)[..., np.newaxis]
    for i in range(size[0]):
        for j in range(size[1]):
            dh = i - hc
            dw = j - wc
            gaus_filter[i, j] = math.exp(-(dh**2 + dw**2) / (2 * sigma1**2))

    # 色についての重みをルックアップテーブルにする
    color_lut = np.zeros((256,))
    for i in range(len(color_lut)):
        color_lut[i] = math.exp(-(i**2) / (2 * sigma2**2))

    tmp_img = img.astype(np.int32)  # プラスマイナスを取れるように型を変換
    padded_img = np.pad(tmp_img, ((hc, hc), (wc, wc), (0, 0)), mode="constant")  # フィルタをかけるためのパディング

    out = np.zeros_like(img)
    for i in range(out.shape[0]):
        for j in range(out.shape[1]):
            diff = color_lut[abs(tmp_img[i, j] - padded_img[i : i + size[0], j : j + size[1]])] # 画素値の差から重みを求める
            w = gaus_filter * diff # 最終的なフィルタを計算する
            out[i, j] = (w * padded_img[i : i + size[0], j : j + size[1]]).sum(axis=(0, 1)) / w.sum(axis=(0, 1))

    return out
```

`size = (15, 15), sigma1 = 20.0, sigma2 = 20.0` での変換例が以下となります。変換前の画像は元の画像に `sigma = 20` のガウシアンノイズをかけたものです。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒ガウシアンノイズ](/images/digital-image-processing/sample1_gray_gausian_noised.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/bilateral_filtering.jpg =250x) |

OpenCVでは `cv2.bilateralFilter(src, d, sigmaColor, sigmaSpace)` でバイラテラルフィルタを行えます。`d` がフィルタサイズ、`sigmaColor` と `sigmaSpace` は上の計算式での$\sigma_2$と$\sigma_1$に当たります。

### ノンローカルミーンフィルタ

バイラテラルフィルタでは、注目画素と周辺画素の画素値の差に応じて重みを決定しました。ノンローカルミーンフィルタでは、注目画素周りの小領域と周辺画素周りの小領域の画素値の差に応じて重みを決定します。式にすると次になります。

$$
w(i, j, m, n) = \exp{\Big(-\frac{\sum_{t=-W'}^{W'}\sum_{s=-W'}^{W'}\big(f(i+s, j+t)-f(i+m+s, j+n+t))^2}{2\sigma^2})}
$$

バイラテラルフィルタと違い、領域の類似度で重みを求めるため、フィルタサイズを大きくしても画像の劣化が生じにくいとされています。

Pythonでの実装例は以下です。

```python
def nonlocal_mean_filtering(img, size1, size2, sigma):
    h1, w1 = size1
    h2, w2 = size2
    hc1, wc1 = h1 // 2, w1 // 2
    hc2, wc2 = h2 // 2, w2 // 2

    tmp_img = img.astype(np.float64)
    padded_img = np.pad(tmp_img, ((hc1, hc1), (wc1, wc1), (0, 0)), mode="constant")  # フィルタをかけるためにパディング

    out = np.zeros_like(img, dtype=np.float64)
    for i in range(out.shape[0]):
        for j in range(out.shape[1]):
            # 注目画素が取りうる範囲
            ref = padded_img[i + hc1 : i + hc1 + h2, j + wc1 : j + wc1 + w2]
            # 注目画素の近傍領域（周辺画素の取りうる範囲）
            search = padded_img[i : i + h1, j : j + w1]
            # 近傍領域の全ての画素それぞれに対する小領域
            patches = sliding_window_view(search, (h2, w2, img.shape[2]))
            diff = np.sum((ref - patches) ** 2, axis=(2, 3, 4)) / (h2 * w2)
            w = np.exp(-diff / (2 * sigma**2)) # 重みを計算する
            out[i, j] = np.sum(w * patches[..., 0, hc2, wc2, :], axis=(0, 1)) / np.sum(w, axis=(0, 1))

    return np.clip(out, 0, np.iinfo(np.uint8).max).astype(np.uint8)
```

上で紹介した計算式を愚直に実装すると4重ループになり、Pythonでは処理が非常に遅くなります。そこで、`sliding_window_view(x, window_shape)` を使用して高速化を図っています。`sliding_window_view(x, window_shape)` はNumpyの関数で、名前の通りスライディングウィンドウを作成できます。`x` がもとの配列で、`window_shape` は各要素に適用するウィンドウの形状となります。

`size1 = (21, 21), size2 = (7, 7), sigma = 10.0` の変換例を以下に示します。変換前の画像はバイラテラルフィルタでも使用したガウシアンノイズをかけた画像です。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒ガウシアンノイズ](/images/digital-image-processing/sample1_gray_gausian_noised.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/nonlocal_mean_filtering.jpg =250x) |

高速化を行いましたが、それでも自分の環境では6分ほど時間がかかりました。

### メディアンフィルタ

ここまでで紹介してきた手法は周辺画素の画素値の（重み付き）平均を求めるものでした。それに対し、メディアンフィルタは領域内の中央値を出力とするフィルタです。メディアンフィルタは、平均を使用するフィルタが苦手なスパイク状のノイズの除去に効果的です。

以下にPythonでの実装例を示します。

```python
def median_filtering(img, size) -> NDArray[np.uint8]:
    h, w = size[0], size[1]
    hc, wc = h // 2, w // 2
    padded_img = np.pad(img, ((hc, hc), (wc, wc), (0, 0)), mode="constant")  # フィルタをかけるためにパディング

    out = np.zeros_like(img)
    for i in range(img.shape[0]):
        for j in range(img.shape[1]):
            out[i, j] = np.median(padded_img[i : i + h, j : j + w], axis=(0, 1))

    return out
```

`size = (3, 3)` での変換例を以下に示します。変換前の画像は元の画像に対し、全体の画素のうち5%の画素値を0または255にしたものです。

| 元画像 | 変換後 |
| ---- | ---- |
| ![サンプル1 白黒スパイクノイズ](/images/digital-image-processing/sample1_gray_spike_noised.jpg =250x) | ![変換後](/images/digital-image-processing/chap5/median_filtering.jpg =250x) |
