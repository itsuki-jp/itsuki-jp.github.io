---
title: Pyxelで作る簡易RPG
date: 2024-12-14T23:42:51+09:00
draft: false
summary: 某アドカレ23日目！！！
---
# はじめに
Pyxelを知っていますか？私は12/01日に日本科学未来館にて行われた「[Scientific Game Jam Tokyo](https://jp-minerals.org/scientificgamejam/jp/)」にて知りました

![alt text](/images/2024-12-23_pyxel_game/image.png)

> [Pyxel](https://github.com/kitao/pyxel/blob/main/docs/README.ja.md)は、Python向けのレトロゲームエンジンです

> 使える色は 16 色のみ、同時に再生できる音は 4 音までなど、レトロゲーム機を意識したシンプルな仕様で、Python を使ってドット絵スタイルのゲームづくりが気軽に楽しめます

Pythonでゲームが作れるフレームワーク？でいうと、以前Kivyを触ったことがあるのですがなかなか曲者だったので、Pyxelはどうでしょうか...???

[Kivyからの挑戦状．Pongを改良しよう！](https://qiita.com/n_itsuki/items/ef654f61c12e27651f6c)

結果としては、Pyxelは（ゲームのアルゴリズムを考えるのを除けば（当然））そこそこ楽にゲームが作れるなぁと感じました

ということで、今回はPyxelを使ってカニ的なRPGを作っていきます！！！

# RPG を作ってみよう

## RPG...???
まず、RPGに必要な要素とは何でしょう？私は基本パズドラ以外のゲームをしてきたことがないので、正直分かりませんただ、イメージとしては
- プレイヤーがいる
- プレイヤーは左右上下に移動できるが、移動できない場所もある
- まれにアイテムが出現し、拾える
- まれに敵が出現し、戦う

かなぁと思っています。

最後の「まれに敵が出現し、戦う」は難しそうなので、それ以外をやっていこうと思います！

## Pyxelを使えるようにする
仮想環境を作成して、`sudo pip3 install -U pyxel` するだけで使えるようになると思っていました。ただ、Windows ではなく、Linux(WSL)で実行するのであれば、追加でSDL2が必要です。

`sudo apt install libsdl2-2.0-0 libsdl2-dev`

これで、pyxelをインストールできるようになります！

## 画面を生成する
次に、画面を生成していきます！

`update()` と`draw()`は特別で、それぞれロジックを書く関数・画面を描写する関数です。この二つの関数に諸々の処理をここから書いていきます。

`pyxel.init(120, 120)`でサイズを指定し、画面を作成しています。

```py
import pyxel

class App:
    def __init__(self):
        pyxel.init(120, 120)

    ##############
    # Game logic #
    ##############
    def update(self):
        return

    ##############
    # Draw logic #
    ##############
    def draw(self):
        return

App()
```

## マップを作成する
フィールドを構成する要素は、
- 草（歩ける）
- 石（通行不可）
- 海（通行不可）
  - 盤外
- アイテム（拾う）

の4つあります。これらを最初に定数として定義し、`map`を作成していきます。

```py
from random import random
GRASS = 0
STONE = 1
SEA = 2
BLOCK_SIZE = 5


class App:
    def __init__(self):
        pyxel.init(120, 120)
        self.map = [
            [
                (
                    GRASS if random() < 0.7 else STONE if random() < 0.9 else ITEM
                )
                for j in range(150)
            ]
            for i in range(150)
        ]

    ##############
    # Draw logic #
    ##############
    def draw(self):
        pyxel.cls(0)
        self.draw_map()

    def draw_map(self):
        for y, row in enumerate(self.map):
            for x, tile in enumerate(row):
                pyxel.rect(
                    x * BLOCK_SIZE,
                    y * BLOCK_SIZE,
                    BLOCK_SIZE,
                    BLOCK_SIZE,
                    title,
                )
```

## プレイヤー中心のマップに変更する
ただ、これだとプレイヤーの位置に依存せず、左上から描写してくだいくだけなので、プレイヤーが中心に表示され、移動すると背景が動くようにしてみます！

<details><summary>長くなってきたコード</summary>

```py
import pyxel
from random import randint, random

GRASS = 0
STONE = 1
SEA = 2
BLOCK_SIZE = 5


class App:
    def __init__(self):
        pyxel.init(120, 120)
        self.x = 50
        self.y = 50
        self.map = [
            [
                (
                    GRASS if random() < 0.7 else STONE if random() < 0.9 else ITEM
                )
                for j in range(150)
            ]
            for i in range(150)
        ]
        pyxel.run(self.update, self.draw)

    ##############
    # Draw logic #
    ##############
    def draw(self):
        pyxel.cls(0)
        self.draw_map()
        self.draw_player()

    def draw_player(self):
        pyxel.rect(
            pyxel.width // 2,
            pyxel.height // 2,
            BLOCK_SIZE,
            BLOCK_SIZE,
            0,
        )

    def draw_map(self):
        start_x = max(self.x - pyxel.width // (2 * BLOCK_SIZE), 0)
        start_y = max(self.y - pyxel.height // (2 * BLOCK_SIZE), 0)
        end_x = min(self.x + pyxel.width // (2 * BLOCK_SIZE), len(self.map[0]))
        end_y = min(self.y + pyxel.height // (2 * BLOCK_SIZE), len(self.map))

        for y, row in enumerate(self.map[start_y:end_y]):
            for x, tile in enumerate(row[start_x:end_x]):
                color = [
                    11,
                    4,
                    5,
                ][tile]
                pyxel.rect(
                    (x + start_x - self.x + pyxel.width // (2 * BLOCK_SIZE))
                    * BLOCK_SIZE,
                    (y + start_y - self.y + pyxel.height // (2 * BLOCK_SIZE))
                    * BLOCK_SIZE,
                    BLOCK_SIZE,
                    BLOCK_SIZE,
                    color,
                )


App()
```

</details>

きもは`draw_map`関数です。

まず、start_x, start_y, end_x, end_yにて描写する範囲を計算し、`pyxel.rect`で描写しています。
特に描写されない部分は最初らへんの`pyxel.cls(0)`にて黒に塗りつぶされているのでOKです

```py
def draw_map(self):
    start_x = max(self.x - pyxel.width // (2 * BLOCK_SIZE), 0)
    start_y = max(self.y - pyxel.height // (2 * BLOCK_SIZE), 0)
    end_x = min(self.x + pyxel.width // (2 * BLOCK_SIZE), len(self.map[0]))
    end_y = min(self.y + pyxel.height // (2 * BLOCK_SIZE), len(self.map))

    for y, row in enumerate(self.map[start_y:end_y]):
        for x, tile in enumerate(row[start_x:end_x]):
            color = [
                11,
                4,
                5,
            ][tile]
            pyxel.rect(
                (x + start_x - self.x + pyxel.width // (2 * BLOCK_SIZE))
                * BLOCK_SIZE,
                (y + start_y - self.y + pyxel.height // (2 * BLOCK_SIZE))
                * BLOCK_SIZE,
                BLOCK_SIZE,
                BLOCK_SIZE,
                color,
            )
```

![alt text](/images/2024-12-23_pyxel_game/image-1.png)

## プレイヤーを動かす
マップが表示されるようになったので、最後はプレイヤーが移動出来たら完璧ですね！

やっと`update()`が必要になります。

`pyxel.btnp(pyxel.KEY_LEFT)`と`pyxel.btnp(pyxel.GAMEPAD1_BUTTON_DPAD_LEFT)`、一見同じように見えますが、ちゃんと役割が異なります。前者はキーボードの左キー、後者はゲームパッドの左キーです。パソコンで開発しているので、よく分からないのですが、タッチデバイスでプレイするときにバーチャルゲームパッドが表示されるとかされないとか。

[Python向けレトロゲームエンジンPyxelがWebに対応しました！](https://tkitao.hatenablog.com/entry/2022/09/26/114014)


<details><summary>長くなってきたコード</summary>

```py
import pyxel
from random import randint, random

GRASS = 0
STONE = 1
SEA = 2
BLOCK_SIZE = 5


class App:
    def __init__(self):
        pyxel.init(120, 120)
        self.x = 50
        self.y = 50
        self.map = [
            [
                (GRASS if random() < 0.7 else STONE if random() < 0.9 else ITEM)
                for j in range(150)
            ]
            for i in range(150)
        ]
        pyxel.run(self.update, self.draw)

    ##############
    # Game logic #
    ##############
    def update(self):
        self.handle_input()

    def handle_input(self):
        new_x = self.x
        new_y = self.y

        if pyxel.btnp(pyxel.KEY_LEFT) or pyxel.btnp(pyxel.GAMEPAD1_BUTTON_DPAD_LEFT):
            new_x -= 1
        elif pyxel.btnp(pyxel.KEY_RIGHT) or pyxel.btnp(
            pyxel.GAMEPAD1_BUTTON_DPAD_RIGHT
        ):
            new_x += 1
        elif pyxel.btnp(pyxel.KEY_UP) or pyxel.btnp(pyxel.GAMEPAD1_BUTTON_DPAD_UP):
            new_y -= 1
        elif pyxel.btnp(pyxel.KEY_DOWN) or pyxel.btnp(pyxel.GAMEPAD1_BUTTON_DPAD_DOWN):
            new_y += 1
        else:
            return
        if self.is_inside_map(new_x, new_y) and self.map[new_y][new_x] != STONE:
            self.x = new_x
            self.y = new_y

    def is_inside_map(self, x, y):
        return 0 <= x < len(self.map[0]) and 0 <= y < len(self.map)


    ##############
    # Draw logic #
    ##############
    def draw(self):
        pyxel.cls(0)
        self.draw_map()
        self.draw_player()
        self.draw_text(x=0, y=0, text=f"Item: {self.item_count}")

    def draw_player(self):
        pyxel.rect(
            pyxel.width // 2,
            pyxel.height // 2,
            BLOCK_SIZE,
            BLOCK_SIZE,
            0,
        )

    def draw_map(self):
        start_x = max(self.x - pyxel.width // (2 * BLOCK_SIZE), 0)
        start_y = max(self.y - pyxel.height // (2 * BLOCK_SIZE), 0)
        end_x = min(self.x + pyxel.width // (2 * BLOCK_SIZE), len(self.map[0]))
        end_y = min(self.y + pyxel.height // (2 * BLOCK_SIZE), len(self.map))

        for y, row in enumerate(self.map[start_y:end_y]):
            for x, tile in enumerate(row[start_x:end_x]):
                color = [
                    11,
                    4,
                    5,
                ][tile]
                pyxel.rect(
                    (x + start_x - self.x + pyxel.width // (2 * BLOCK_SIZE))
                    * BLOCK_SIZE,
                    (y + start_y - self.y + pyxel.height // (2 * BLOCK_SIZE))
                    * BLOCK_SIZE,
                    BLOCK_SIZE,
                    BLOCK_SIZE,
                    color,
                )


App()
```

</details>


## アイテム要素
これで、一応それっぽいのは出来たはずです！

ただ、ポケモンにて草むらにアイテムがあるように、このRPGにもアイテムが欲しいですね！これはマップで隠しアイテムフラグ？を持っておけば解決できます。楽でよいですね！！！

<details><summary>完成コード</summary>

```py
import pyxel
from random import randint, random

GRASS = 0
STONE = 1
SEA = 2
ITEM = 3
BLOCK_SIZE = 5


class App:
    def __init__(self):
        pyxel.init(120, 120)
        self.x = 50
        self.y = 50
        self.item_count = 0
        self.map = [
            [
                (GRASS if random() < 0.7 else STONE if random() < 0.9 else ITEM)
                for j in range(150)
            ]
            for i in range(150)
        ]
        pyxel.run(self.update, self.draw)

    ##############
    # Game logic #
    ##############
    def update(self):
        self.handle_input()

    def handle_input(self):
        new_x = self.x
        new_y = self.y

        if pyxel.btnp(pyxel.KEY_LEFT) or pyxel.btnp(pyxel.GAMEPAD1_BUTTON_DPAD_LEFT):
            new_x -= 1
        elif pyxel.btnp(pyxel.KEY_RIGHT) or pyxel.btnp(
            pyxel.GAMEPAD1_BUTTON_DPAD_RIGHT
        ):
            new_x += 1
        elif pyxel.btnp(pyxel.KEY_UP) or pyxel.btnp(pyxel.GAMEPAD1_BUTTON_DPAD_UP):
            new_y -= 1
        elif pyxel.btnp(pyxel.KEY_DOWN) or pyxel.btnp(pyxel.GAMEPAD1_BUTTON_DPAD_DOWN):
            new_y += 1
        else:
            return
        if self.is_inside_map(new_x, new_y) and self.map[new_y][new_x] != STONE:
            self.x = new_x
            self.y = new_y
            print("hogehoge!")

            if self.is_get_item(new_x, new_y):
                self.item_count += 1

    def is_inside_map(self, x, y):
        return 0 <= x < len(self.map[0]) and 0 <= y < len(self.map)

    def is_get_item(self, x, y):
        return self.map[y][x] == ITEM

    ##############
    # Draw logic #
    ##############
    def draw(self):
        pyxel.cls(0)
        self.draw_map()
        self.draw_player()
        self.draw_text(x=0, y=0, text=f"Item: {self.item_count}")

    def draw_player(self):
        pyxel.rect(
            pyxel.width // 2,
            pyxel.height // 2,
            BLOCK_SIZE,
            BLOCK_SIZE,
            0,
        )

    def draw_text(self, x, y, text):
        pyxel.text(x, y, text, 1)

    def draw_map(self):
        start_x = max(self.x - pyxel.width // (2 * BLOCK_SIZE), 0)
        start_y = max(self.y - pyxel.height // (2 * BLOCK_SIZE), 0)
        end_x = min(self.x + pyxel.width // (2 * BLOCK_SIZE), len(self.map[0]))
        end_y = min(self.y + pyxel.height // (2 * BLOCK_SIZE), len(self.map))

        for y, row in enumerate(self.map[start_y:end_y]):
            for x, tile in enumerate(row[start_x:end_x]):
                color = [
                    11,
                    4,
                    5,
                    11,
                ][tile]
                pyxel.rect(
                    (x + start_x - self.x + pyxel.width // (2 * BLOCK_SIZE))
                    * BLOCK_SIZE,
                    (y + start_y - self.y + pyxel.height // (2 * BLOCK_SIZE))
                    * BLOCK_SIZE,
                    BLOCK_SIZE,
                    BLOCK_SIZE,
                    color,
                )


App()
```
</details>

## おまけ
せっかくコードを書いたので、実際に人々にプレイしてもらいたいですよね（？）[公開方法は以下の2種類](https://github.com/kitao/pyxel/blob/main/docs/pyxel-web-ja.md)あります。

### 1. Pyxel Web Launcher に GitHub リポジトリを指定する

`https://kitao.github.io/pyxel/wasm/launcher/?<コマンド>=<githubのユーザー名>.<リポジトリ名>.<アプリのディレクトリ>.<拡張子を取ったファイル名>`

GitHubのレポジトリが公開されていれば、Launcherからプレイ出来るそうです

### 2. Pyxel アプリを HTML ファイルに変換する

`pyxel app2html your_app.pyxapp`

これでpuxappファイルをHTMLに変換して、それを公開出来るそうです


今回はせっかくなので、HTMLに変換してみます！
私の作業環境には`main.py`しかないので、`pyxel package . ./main.py `で コードをpyxappに変換し、`pyxel app2html pyxel_rpg.pyxapp `でHTMLに変換しました。

[(複数ファイルがある場合は、`pyxel package アプリケーションフォルダ 起動スクリプトファイル`のように実行するらしい)](https://kinutani.hateblo.jp/entry/2023/02/10/184212)

このHTMLをGitHub Pagesに上げれば完璧ですね！！！

[Pyxel RPG](https://itsuki-jp.github.io/pyxel_rpg/)

# さいごに
ということで、今回はPyxelでRPGもどきを作ろうの回でした。今回はM1に熱中し時間が足らずPyxelの様々な昨日（画像・タイルマップ・音楽エディター）などを使えなかったのですが、再来月あたりにブラッシュアップしたものを上げられれば良いなぁと希望を抱いてます。