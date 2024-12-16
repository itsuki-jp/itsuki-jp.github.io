---
title: HTMLとJavaScriptでゲームを作りたい！
date: 2024-12-03T21:57:35+09:00
draft: false
summary: "make game using javascript"
---
# はじめに
みなさん、ゲームは好きですか？自分で好きなゲームを作れたら楽しそうじゃないですか！？この記事は「ゲーム作るのはハードルが高そうでなんか難しそうだし...」と一歩が踏み出せない人間向けの記事です。

一言で「ゲームを作る」とはいっても、いろいろなやり方があります。有名どころだと、
- C#で書くUnity
- C++で書くUnreal Engine
- 
があります。ただ、これらはプログラミング言語の知識に加えて、諸々の画面操作などの知識も必要になり、まあまあめんどくさいです。

今回は、プログラミングの知識があれば簡単に作れる、HTMLとJavaScriptを使ってゲームを作っていこうと思います。


# 作ってみよう
## 作るもの
https://itsuki-jp.github.io/rhythmGameUsingJavaScript/

今回解説するのは、音ゲー（音がないので落ちてくるブロックをタイミングよくクリックする）っぽいゲームです。
![alt text](/images/2024-12-16_make_game_using_javascript/image-1.png)

## canvas解説
### canvas
HTML+JavaScriptでゲームを作るのに欠かせない要素として、`canvas`があります。

> HTML の \<canvas\> 要素 と キャンバススクリプティング API や WebGL API を使用して、グラフィックやアニメーションを描画することができます。

https://developer.mozilla.org/ja/docs/Web/HTML/Element/canvas


結構前に[canvasの使い方（備忘録）](https://qiita.com/n_itsuki/items/563fe0dc84565ddc324e)で基本的な使い方を書いたのですが、今回は音ゲー（）の解説とともに進めていきます。

### 最初の準備
まずはHTMLとJavaScriptを用意して、それらを紐づける必要があります。

`idnex.html`
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <canvas id="canvas"></canvas>
    <script src="./index.js"></script>
</body>

</html>
```

`index.js`
```javascript
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");
```

あとは、`ctx`を適当に操作していくと、図形を描写したり、文字を書いたりの諸々ができるようになります。

### 四角を描写する
このゲームにおいて、最も重要な形は四角です。ノードも、黒い線も、判定エリアもすべて四角です。一番下に、一番最初のノードを描写する例があります。
```javascript
const colors = ["red", "blue", "yellow", "purple", "orange"];
const NODES = Array.from({ length: 20 }, (_, i) => ({
    line: Math.floor(Math.random() * 5),
    time: (i + 1) * 10,
    color: colors[Math.floor(Math.random() * 5)],
    size: nodeSize,
    checked: false,
}))

// 四角を描画する関数
// 引数：書きたい四角、色、透明度（書かなくてもOK）
const drawRect = (rect, color, alpha = 1) => {
    ctx.globalAlpha = alpha;
    ctx.fillStyle = color;
    ctx.fillRect(rect.pos.x, rect.pos.y, rect.size.x, rect.size.y);
    ctx.globalAlpha = 1;
}

// 例
drawRect(NODES[0], NODES[0].color);
```

### 文字を描写する
四角ほど重要ではないですが、スコアとかを表示できたほうが楽しいですよね！
```javascript
// テキストを描画する関数
const drawText = (text, x, y, font, color) => {
    ctx.fillStyle = color;
    ctx.font = font;
    ctx.fillText(text, x, y);
}
```

## その他のそこそこ重要な関数
### キー入力
このゲームはパソコンで遊ぶことを前提に作られているので、キーボードの入力で諸々を判定しています
```javascript
// キーが押されたときの処理
document.addEventListener("keydown", (e) => {
    if (e.key === "a") {
        CURRENT_NODES.forEach((node) => {
            score += isCollided(node, checkArea[0]) ? 1 : 0;
        });
    }
    else if (e.key === "s") {
        CURRENT_NODES.forEach((node) => {
            score += isCollided(node, checkArea[1]) ? 1 : 0;
        });
    }
    else if (e.key === "d") {
        CURRENT_NODES.forEach((node) => {
            score += isCollided(node, checkArea[2]) ? 1 : 0;
        });
    }
    else if (e.key === "f") {
        CURRENT_NODES.forEach((node) => {
            score += isCollided(node, checkArea[3]) ? 1 : 0;
        });
    }
    else if (e.key === "g") {
        CURRENT_NODES.forEach((node) => {
            score += isCollided(node, checkArea[4]) ? 1 : 0;
        });
    }
}
)
```

### 画面の更新
ノードを落としたりするためには、画面を更新する必要があります。具体的には、
1. 画面を緑色で塗りつぶす
2. 線を描写する
3. 判定エリアを描写する
4. ノードを描写する

\+ キー入力があり、判定エリア内にあればスコア表示を増やす
の操作が必要です
```javascript
// 10msごとに実行する処理
setInterval(() => {
    // 盤面を初期化する
    ctx.fillStyle = "green";
    ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
    LINES.forEach((line) => {
        drawRect(line, "black");
    });
    checkArea.forEach((area) => {
        drawRect(area, "gray", 0.7);
    });
    // ノードを追加する
    for (let i = 0; i < NODES.length; i++) {
        if (NODES[i].time <= now / 10) {
            const newNode = NODES.shift();
            newNode.pos = { x: offset + lineWidth + newNode.line * lineSpace, y: 0 };
            CURRENT_NODES.push(newNode);
        } else {
            break;
        }
    }
    // ノードを動かして、描写する
    CURRENT_NODES.forEach((node) => {
        // ノードが盤外に移動したら削除する
        if (node.pos.y + node.size.y > CANVAS_HEIGHT) {
            // この処理だと、正確にコンボは計測できないけど、いったんね！
            if (node.checked) {
                combo++;
            } else {
                combo = 0;
            }
            CURRENT_NODES.shift();
            return;
        }
        moveNode(node);
        drawRect(node, node.color);
    });
    drawText("score: " + score, 10, 30, "30px sans-serif", "white");
    drawText("combo: " + combo, 10, 60, "30px sans-serif", "white");
    now++;
    console.log(score)
}
    , 10);
```

# まとめ
最終的な、ほかにも必要なコードを付け加えると、以下のようになります（169行、短いですね！！！）
<details><summary>index.js</summary>

```javascript
// =====================
// 必要な変数を定義
// =====================

// 定数の定義
const CANVAS_HEIGHT = 1000;
const CANVAS_WIDTH = 1000;
const offset = 200;
const lineSpace = 110;
const lineWidth = 10;
const nodeSize = { x: 100, y: 40 };
// 落ちてくるノードの定義
const colors = ["red", "blue", "yellow", "purple", "orange"];
const NODES = Array.from({ length: 20 }, (_, i) => ({
    line: Math.floor(Math.random() * 5),
    time: (i + 1) * 10,
    color: colors[Math.floor(Math.random() * 5)],
    size: nodeSize,
    checked: false,
}))
// ラインの定義
const LINES = Array.from({ length: 6 }, (_, i) => ({
    line: i,
    size: { x: lineWidth, y: CANVAS_HEIGHT },
    pos: { x: offset + lineSpace * i, y: 0 }
}));
// 判定エリアの定義
const checkArea = Array.from({ length: 5 }, (_, i) => ({
    line: i,
    size: nodeSize,
    pos: { x: offset + lineWidth + lineSpace * i, y: CANVAS_HEIGHT - 100 }
}));
// 現在落ちてきてるノードの定義
const CURRENT_NODES = [];
// スコアの定義(判定エリアにノードがあるときに、特定のキーを押したら+1)
let score = 0;

// canvasをJavaScriptで使えるようにする
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

// canvasの初期設定
canvas.height = 1000;
canvas.width = 1000;

// =====================
// メインの処理
// =====================

// 四角を描画する関数
// 引数：書きたい四角、色、透明度（書かなくてもOK）
const drawRect = (rect, color, alpha = 1) => {
    ctx.globalAlpha = alpha;
    ctx.fillStyle = color;
    ctx.fillRect(rect.pos.x, rect.pos.y, rect.size.x, rect.size.y);
    ctx.globalAlpha = 1;
}

// ノードを動かす関数
// 引数：動かしたいノード
const moveNode = (node) => {
    node.pos.y += 5;
}

// ノードと判定エリアが衝突しているかどうかを判定する関数
// 引数：ノード、判定エリア
const isCollided = (node, area) => {
    // ノードが判定エリアのラインにあるか確認
    if (node.line !== area.line) return false;
    // 既に判定してたら無視する
    if (node.checked) return false;
    // ノードが判定エリアにある場合は、ノードを確認済みにする
    const nodeTop = node.pos.y;
    const nodeBottom = node.pos.y + node.size.y;
    const areaTop = area.pos.y;
    const areaBottom = area.pos.y + area.size.y;
    if (nodeBottom > areaTop && nodeTop < areaBottom) {
        node.checked = true;
        return true;
    }
    return false;
}

// テキストを描画する関数
const drawText = (text, x, y, font, color) => {
    ctx.fillStyle = color;
    ctx.font = font;
    ctx.fillText(text, x, y);
}

// キーが押されたときの処理
document.addEventListener("keydown", (e) => {
    if (e.key === "a") {
        CURRENT_NODES.forEach((node) => {
            score += isCollided(node, checkArea[0]) ? 1 : 0;
        });
    }
    else if (e.key === "s") {
        CURRENT_NODES.forEach((node) => {
            score += isCollided(node, checkArea[1]) ? 1 : 0;
        });
    }
    else if (e.key === "d") {
        CURRENT_NODES.forEach((node) => {
            score += isCollided(node, checkArea[2]) ? 1 : 0;
        });
    }
    else if (e.key === "f") {
        CURRENT_NODES.forEach((node) => {
            score += isCollided(node, checkArea[3]) ? 1 : 0;
        });
    }
    else if (e.key === "g") {
        CURRENT_NODES.forEach((node) => {
            score += isCollided(node, checkArea[4]) ? 1 : 0;
        });
    }
}
)
// 今の秒数的な
let now = 0;
let combo = 0;

// =====================
// 10msごとに実行する処理
// =====================

setInterval(() => {
    // 盤面を初期化する
    ctx.fillStyle = "green";
    ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
    LINES.forEach((line) => {
        drawRect(line, "black");
    });
    checkArea.forEach((area) => {
        drawRect(area, "gray", 0.7);
    });
    // ノードを追加する
    for (let i = 0; i < NODES.length; i++) {
        if (NODES[i].time <= now / 10) {
            const newNode = NODES.shift();
            newNode.pos = { x: offset + lineWidth + newNode.line * lineSpace, y: 0 };
            CURRENT_NODES.push(newNode);
        } else {
            break;
        }
    }
    // ノードを動かして、描写する
    CURRENT_NODES.forEach((node) => {
        // ノードが盤外に移動したら削除する
        if (node.pos.y + node.size.y > CANVAS_HEIGHT) {
            // この処理だと、正確にコンボは計測できないけど、いったんね！
            if (node.checked) {
                combo++;
            } else {
                combo = 0;
            }
            CURRENT_NODES.shift();
            return;
        }
        moveNode(node);
        drawRect(node, node.color);
    });
    drawText("score: " + score, 10, 30, "30px sans-serif", "white");
    drawText("combo: " + combo, 10, 60, "30px sans-serif", "white");
    now++;
    console.log(score)
}
    , 10);

```

</details>

<details><summary>index.html</summary>

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <canvas id="canvas"></canvas>
    <script src="./index.js"></script>
</body>

</html>
```

</details>

# 最後に
私が音ゲー（）が下手なのか、ロジックが間違っているのかわかりませんが、判定がうまくできないような気がします...間違いを見つけたら教えてくださいorz

ちなみにですが、大学時代はパソコン部的なのに入っており、ゲームを作る必要があったUnityを初めて触りました。まあ難しかったですね。一番最初に作ったゲームは、変数の概念もよくわかってないレベルなので、画面外に変数用のブロックを置いて、毎度そこからデータを取得するようなよくわからない書き方をしていました...
