---
title: "【Web App】マインスイーパー"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python", "Reflex"]
published: true
---

## はじめに

新しいWeb上のゲームとしてマインスイーパーを実装し、公開しました。
単純に遊びたい方は以下にアクセスしてください（デサインは最低限しかいじってないので目をつぶってもらえると助かります）。動作に不具合などがあれば教えて下さい。

:::message
自身の実装の問題か不明ですが、非常に動作が遅いです。ローカルで実行するほうが快適に遊べます。
:::

@[card](https://web_games_app-blue-moon.reflex.run/minesweaper)

## コード

マインスイーパーを含むゲームのコードは以下となっています。

https://github.com/maki8maki/WebGames

## 実装

実装はPythonとReflexで行いました。

### 盤面の生成

盤面の生成用のコードを以下に示します。
簡単のために、最初に選択した箇所の周囲に地雷を設置しないようにしています。地雷の位置を決定後に周囲の地雷数も盤面に記録します。

```python
def initialize(self, num: int):
    """
    選択した数字に応じてセルを初期化する。最初に選択した数字の周囲は地雷が設置されない。

    Args:
        num (int): 選択した数字
    """
    # 選択したマスの周囲を除いて地雷の位置を決める
    excluded_nums = self.get_surroundings(num)
    candidates = [i for i in range(self.num_cells) if i not in excluded_nums]
    mines_nums = rnd.sample(candidates, self.num_mines)
    self.actual_board[self.num2index(mines_nums)] = MINE_NUM

    # 周囲の地雷の数を数える
    for i in range(self.height):
        for j in range(self.width):
            if self.actual_board[i, j] == MINE_NUM:
                continue
            surroundings = self.get_surroundings(self.index2num(i, j))
            sum = 0
            for n in surroundings:
                if self.actual_board[self.num2index(n)] == MINE_NUM:
                    sum += 1
            self.actual_board[i, j] = sum

    self.is_initialized = True
```

### セルの選択・旗の設置と撤去

セルの選択・旗の設置と撤去のコードを以下に示します。
セルを開けるときに地雷かどうかを判断し、地雷だった場合は失敗となるためFalseを返します。地雷でないときは周囲の地雷数を表示します。また、周囲に地雷がないマスだった場合はさらに周囲のマスを開ける候補にしていきます。
旗の設置と撤去については表示用の盤面の変化のみで行っています。

```python
def open_cell(self, num: int) -> bool:
    """
    選択されたセルを開ける

    Args:
        num (int): 選択した数字

    Returns:
        bool: 地雷を選択したときのみFalseを返す
    """
    if self.actual_board[self.num2index(num)] == MINE_NUM:
        return False
    else:
        if not self.is_initialized:
            self.initialize(num)
        q = deque([num])
        while len(q) > 0:
            n = q.pop()
            idx = self.num2index(n)
            if self.is_selected(idx):
                # 旗が置かれているか選択済みのときにスキップ
                continue
            elif self.actual_board[idx] != MINE_NUM:
                self.showing_board[idx] = self.actual_board[idx]  # 表示値を更新
                self.num_selected_cells += 1  # 選択済みセルの数を更新
                if self.actual_board[idx] == 0:
                    # 周囲のセルも開ける候補にする
                    surroundings = self.get_surroundings(n)
                    for s in surroundings:
                        if not self.is_selected(s):
                            q.append(s)
        return True

def put_or_unput_flag(self, num: int):
    """
    選択されたセルに旗を設置または排除する

    Args:
        num (int): 選択された数字
    """
    idx = self.num2index(num)
    if self.showing_board[idx] == FLAG_NUM:
        self.showing_board[idx] = NOT_SELECTED_NUM
    elif self.showing_board[idx] == NOT_SELECTED_NUM:
        self.showing_board[idx] = FLAG_NUM
```

### Webページ

Webページは[Reflex](https://reflex.dev/)というライブラリを用いて実装しています。Reflexを使うとPythonのみでWebページの作成ができます。

#### 表示

表示は以下のようになっています。

セルを左クリックしたときに開ける、右クリックしたときに旗の設置・撤去する（=上記の関数を呼び出す）ように実装しました。旗の数や経過時間については、Reflexのフロントエンド・バックエンド側で管理しています。

![Mine Sweaper](/images/web-minesweaper/minesweaper.png =300x)

#### 記録

記録は以下のようになっています。
記録はReflexのデータベース機能を使用しています。

![Records](/images/web-minesweaper/records.png =150x)

こちらが新たなデータの追加のコードです。
記録するデータ難易度（高さ・幅・地雷数）とクリアタイムのみです。データの追加の後に11番目以降のデータの削除もしています。

```python
def update_record(self) -> int:
    state = to_state(height=self.height, width=self.width, num_mines=self.num_mines)
    with rx.session() as session:
        session.add(MSRecord(state=state, time=self.elapsed_time))
        session.commit()

        records = session.exec(
            MSRecord.select().where(MSRecord.state == state).order_by(MSRecord.time.asc())
        ).all()
        for i in range(10, len(records)):
            session.delete(records[i])
            session.commit()
```

こちらが記録の表示のコードです。
データの抽出をした後に、順位付けをしています。同じタイムのときは同じ順位となるようにしています。

```python
@rx.var(cache=False)
def data(self) -> List[List[int]]:
    with rx.session() as session:
        records = session.exec(
            MSRecord.select().where(MSRecord.state == self.state).order_by(MSRecord.time.asc())
        ).all()

    data = []
    for i in range(min(10, len(records))):
        time = records[i].time
        if i > 0 and time == records[i - 1].time:
            rank = data[-1][0]
        else:
            rank = i + 1
        data.append([rank, time])

    return data
```

## おわりに

マインスイーパーの基礎的な実装についてはそこまで苦労することはありませんでした。一方で、Reflexとの連携面で調整することが多かったです。また、Reflexのデータベース機能も初めて利用したため、試行錯誤しました。

機会があれば新しいゲームの実装もしたいと思います。アイデアがあればお待ちしています。

## 参考

- [Reflexドキュメント](https://reflex.dev/)
